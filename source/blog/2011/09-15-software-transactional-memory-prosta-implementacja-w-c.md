@{
    Layout = "post";
    Title = "Software Transactional Memory - prosta implementacja w C#";
    Date = "2011-09-15T17:50:12+00:00";
    Tags = "";
    Description = "";
}

Ostatnio zderzyłem się z problemem transakcyjnej pamięci (STM) i właściwie brakiem dostępnych implementacji w C# (poza [NSTM](http://code.google.com/p/nstm/ "NSTM")). Różne wariacje na temat które powstały przez lata w stajni Microsoftu nie doczekały się do dnia dzisiejszego funkcjonującej implementacji (głównie dlatego, że zdecydowano - całkiem rozsądnie - że wsparcie dla tego mechanizmu powinno być na poziomie CLR). Dla osób zainteresowanych tematem przedstawię w skrócie moją implementację, zapewne nie pozbawioną wad, jednak mającą jedną zasadniczą zaletę - prostotę implementacji i użycia. Zatem zapraszam do lektury :-)
  
<!--more-->
  
Ponieważ zakłada się, że przez większość czasu system (program) powinien działać prawidłowo, przyjąłem optymistyczne podejście do transakcji. Zatem cały mechanizm polega na tym, iż tworzymy w pewnym momencie (o czym już sami decydujemy) kopię danych obiektu, którą w razie potrzeby odtwarzamy (rollback). Przede wszystkim trzeba więc zacząć od sposobu zachowywania kopii obiektu do ewentualnego odtworzenia. Ja zdecydowałem się na mechanizm serializacji, głównie ze względu na sporą uniwersalność. Alternatywnie można wykorzystać np. klonowanie (IClonable i MemberwiseClone()), co jest zwykle znacznie szybsze, jednak wymaga od każdego zapisywanego obiektu realizacji IClonable. W przypadku serializacji wystarczające jest oznaczenie obiektu jako Serializable, co zwykle jest proste do osiągnięcia. Kolejnym ograniczeniem, które przyjąłem, jest odtwarzanie tylko i wyłącznie publicznych właściwości obiektu - zakładamy, że operujemy na stosunkowo prostych obiektach (typu nośniki danych z bazy), co pozwala nam znacznie uprościć realizację wycofywania transakcji.
  
Odtwarzanie stanu obiektów oparte jest o refleksję - przepisujemy wartości właściwości z kopii do obiektu źródłowego (nie robimy podmiany na zdeserializowany obiekt - zmieniło by to referencję, czego chciałem uniknąć). Daje to niewielki narzut wydajnościowy, jednak przy optymistycznej transakcji strata w rollbacku nie jest aż takim problemem (przynajmniej w moim przypadku).
  
Zatem założenia znamy, pora poznać implementację. Oto i ona (C# 4.0):

    public class MemoryTransaction : IDisposable
    {
        #region Static members

        private static ConcurrentDictionary&lt;int, ConcurrentStack&lt;MemoryTransaction&gt;&gt; transactionStack = new ConcurrentDictionary&lt;int, ConcurrentStack&lt;MemoryTransaction&gt;&gt;();
        private static object locker = new object();

        public static MemoryTransaction Current
        {
            get
            {
                if (transactionStack.ContainsKey(Thread.CurrentThread.ManagedThreadId) && transactionStack[Thread.CurrentThread.ManagedThreadId].Count &gt; 0)
                {
                    MemoryTransaction result;
                    transactionStack[Thread.CurrentThread.ManagedThreadId].TryPeek(out result);
                    return result;
                }
                else
                    return null;
            }
        }

        #endregion

        #region Private members

        private Stack&lt;object&gt; myTransactionLogObjects;
        private Stack&lt;byte[]&gt; myTransactionLogStates;

        #endregion

        #region Constructors

        public MemoryTransaction()
        {
            var stack = transactionStack.GetOrAdd(Thread.CurrentThread.ManagedThreadId, new ConcurrentStack&lt;MemoryTransaction&gt;());
            transactionStack[Thread.CurrentThread.ManagedThreadId].Push(this);
            myTransactionLogObjects = new Stack&lt;object&gt;();
            myTransactionLogStates = new Stack&lt;byte[]&gt;();
        }

        #endregion

        public void SaveObject(object instance)
        {
            if ((null != instance) && (false == instance is ValueType) && (instance.GetType().IsSerializable))
            {
                if (false == myTransactionLogObjects.Contains(instance))
                    lock (locker)
                    {
                        myTransactionLogObjects.Push(instance);
                        myTransactionLogStates.Push(instance.Serialize());
                    }
            }
            else
            {
                throw new ArgumentException("Provide existing instance of Serializable class");
            }
        }

        private void RecoverObject(object instance, object copy)
        {
            Type t = instance.GetType();
            PropertyInfo[] propInfos = t.GetProperties();
            lock (locker)
            {
                foreach (PropertyInfo info in propInfos)
                    if (info.CanWrite && info.PropertyType.IsSerializable)
                    {
                        info.SetValue(instance, info.GetValue(copy, null), null);
                    }
            }
        }

        public void Commit()
        {
            lock (locker)
            {
                myTransactionLogObjects.Clear();
                myTransactionLogStates.Clear();
            }
        }

        public void Dispose()
        {
            ConcurrentStack&lt;MemoryTransaction&gt; stack;
            MemoryTransaction current;
            transactionStack[Thread.CurrentThread.ManagedThreadId].TryPop(out current);
            // Clean transaction stack for current thread if empty
            if (transactionStack[Thread.CurrentThread.ManagedThreadId].Count == 0)
                transactionStack.TryRemove(Thread.CurrentThread.ManagedThreadId, out stack);
            // Restore state for all saved objects
            while (myTransactionLogObjects.Count &gt; 0)
            {
                RecoverObject(myTransactionLogObjects.Pop(), myTransactionLogStates.Pop().Deserialize());
            }
        }
    }


Jak widać, ilość kodu nie powala na łopatki, stopień skomplikowania również. Dodatkowo wykorzystywana jest dodatkowa klasa pomocnicza, która dodaje dwie metody rozszerzające (Serialize na object i Deserialize na byte[]) które realizują (de)serializację binarną (System.Runtime.Serialization.Formatters.Binary). Użycie transakcji wygląda identycznie jak TransactionScope (więc można w prosty sposób powiązać STM z transakcją na bazie danych - wystarczą tylko drobne modyfikacje powyższej klasy):

    using (MemoryTransaction mt = new MemoryTransaction())
    {
        // Tutaj robimy nasz misz-masz...

        // Commit robimy jeżeli wszystko OK, w przeciwnym wypadku automatyczne wykonany zostanie Rollback
        mt.Commit();
    }


Zostaje jeszcze samo zapisywanie obiektów. Można to zrobić co najmniej na dwa sposoby - albo jawnie w bloku transakcji, wywołując `mt.SaveObject(naszObiekt)`, albo na poziomie obiektu podczas ustawiania wartości jego pól (np. przy zmianie znacznika typu IsDirty, a właściwie przed jego zmianą) - wywołujemy wtedy `MemoryTransaction.Current.SaveObject(this);`. Jak widać nie jest to bardzo uciążliwe, należy jednak zawsze zadbać o "ręczny" zapis stanu obiektu - inaczej rollback nic nie zmieni.
  
Jedną z pozytywnych cech poniższej implementacji jest względne bezpieczeństwo w przypadku aplikacji wielowątkowej (thread safety). Każdy z wątków posiada własny stos transakcji, więc raczej nie ma możliwości "przeplatania się" transakcji. Ponadto krytyczne zapisy i odczyty wykonywane są podczas locka - co niestety ujemnie wpływa na wydajność, ale bezpieczeństwo jest w tym wypadku najważniejsze.
  
Inną pozytywną cechą jest możliwość tworzenia zagnieżdżonych transakcji - więc nie ma niebezpieczeństwa "wysypania" transakcji poprzez np. wywołanie metody która uruchamia własną transakcję. Ważnym aspektem tego zachowania jest kwestia rejestracji obiektu nie tylko w bieżącej, ale również w nadrzędnych transakcjach - co w podanej implementacji nie jest zaimplementowane, jednak nie jest trudne do osiągnięcia.

Zatem jak widać możliwe jest stworzenie mechanizmów zbliżonych do "prawdziwego" STM, które nie utrudnią nam za bardzo życia i pozwolą na utworzenie względnie bezpiecznej aplikacji bez powtarzania niepotrzebnych fragmentów kodu. Jak każde uproszczenie ma swoje zalety i wady - do tych drugich należy zapewne ograniczone zastosowanie i niekorzystny wpływ na wydajność. Spadek wydajności w stosunku do niestosowania transakcji w przypadku niezbyt skomplikowanych obiektów (nie zawierających złożonych obiektów typu listy) sięga 30-50% dla optymistycznej, pozytywnie zakończonej operacji. Czy jest to dużo, czy mało - to już rzecz typowo subiektywna - wszystko zależy od konkretnych potrzeb. Oczywiście nie mogę zagwarantować, że implementacja ta jest w 100% poprawna i zawsze zadziała bez zarzutu - jednak w moich testach jeszcze mnie nie zawiodła i miejmy nadzieję, że będzie tak dalej :-)