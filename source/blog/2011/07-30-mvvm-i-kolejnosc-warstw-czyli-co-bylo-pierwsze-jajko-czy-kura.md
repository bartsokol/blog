@{
    Layout = "post";
    Title = "MVVM i kolejność warstw, czyli co było pierwsze - jajko czy kura?";
    Date = "2011-07-30T10:46:50+00:00";
    Tags = "";
    Description = "";
}

Mała zmiana tematu po kilku miesiącach - opowiem dziś coś niecoś o technologiach ze stajni Microsoftu i związanych z nimi wzorcami. A tak konkretniej to będzie to (bodajże najpopularniejszy) wzorzec związany z Silverlight i Windows Presentation Foundation (WPF) - czyli MVVM. Opowiem jaka powinna być (moim skromnym zdaniem) kolejność tworzenia poszczególnych warstw, co postaram się poprzeć "dowodami", nie koniecznie formalnymi. Opowiem również o związanych z implementacją wzorca problemach i być może nawet o rozwiązaniach tych problemów.
<!--more-->

### Co powinno być pierwsze - view czy view model?

Odpowiedź jest jedna, czasem podważana przez niektórych przedstawicieli programistycznego świata. Otóż brzmi ona tak: View. A dlaczegóż tak?

Zacznę może od poprawnej nazwy wzorca MVVM - powinna ona brzmieć VVMM, czyli View -> ViewModel -> Model. Dlaczego taka kolejność jest tak ważna? Otóż wszystko wynika ze słowa - klucza związanego z tym i innymi wzorcami - separacji. Każda z tych warstw powinna być możliwie najluźniej związana z warstwą o poziom niższą, natomiast o warstwie "2 dalej" nie powinna nic wiedzieć (dotyczy to szczególnie widoku). Ważny jest również kierunek strzałek - określa on kierunek powstawania kolejnych warstw. Każda strzałka daje również możliwość przecięcia - łącznie z możliwością całkowitego wyeliminowania powiązań między warstwami. Osiągnięcie tego jest jak najbardziej możliwe i wcale nie tak trudne, jak się z początku wydaje. Korzystając z jednego z wielu dostępnych frameworków wspierających MVVM można w dość krótkim czasie skonstruować elegancką aplikację, która w dodatku będzie miała przejrzysty i czytelny kod.

### Narzędzia

Jednym z takich "wspomagaczy" jest dostarczany przez zespół Microsoft Patterns & Practices framework Prism (wcześniej zwany również jako Composite Application Guidance) wraz z bibliotekami IoC (Inversion of Control) Unity lub MEF. Dostarcza on zestaw niezbędnych klas i interfejsów (np. commands, behaviours, regions), które ułatwiają budowę aplikacji nie tylko zgodnej z MVVM, ale również modularnej, składającej się z luźno powiązanych ze sobą projektów. Jeżeli tworzymy rozbudowaną aplikację biznesową to warto się przyjrzeć temu rozbudowanemu narzędziu. Oczywiście dostępne są również inne biblioteki (np. bardzo ciekawy MVVM Light Toolkit) i wybór konkretnej z nich jest kwestią potrzeb i preferencji związanych z konkretnym projektem.

### A dlaczego tak? cz. 1 - blendability

Wracając do podziału na warstwy i ich kolejności - dlaczego taka kolejność ma takie znaczenie? Odpowiedź może nie wydawać się oczywista, ale nie wymaga żadnej tajemnej wiedzy, aby ją zrozumieć i zastosować. Po pierwsze - dlaczego widok powinien być niezależny od view modela, ale jednocześnie tworzyć jego instancję? (i to w code behind!)

Taka kolejność wynika ze ścisłego związku ze strukturą aplikacji w WPF/SL. Jako że widok to w 95% deklaratywny kod XAML (pozostałe 5% to fragment tworzący VM w code behind oraz kod klasy bazowej widoków), który może być tworzony zarówno w Visual Studio, Expression Blend jak i innych narzędziach (jak np. Kaxaml czy Notepad :-) ), nie było by rozsądnym tworzenie i wiązanie go z poziomu kodu logiki aplikacji. Znacząco utrudniało by to (lub wręcz uniemożliwiało) projektowanie interfejsu niezależnie od logiki działania aplikacji. Przy odpowiedniej separacji mamy możliwość wygenerowania danych testowych i projektowania niezależnie od implementacji (której może w ogóle jeszcze nie być), co w przypadku Expression Blend jest wyjątkowo proste i wygodne. Dodając do tego ignorowanie code behind przez Blenda mamy pierwszą część odpowiedzi, która w dodatku zostanie poparta przez następną część.

### A dlaczego tak? cz.2 - testability

Mając widok, który w razie potrzeby tworzy sobie (prawdziwy lub mockowany przez designera) view model, jesteśmy w stanie całkowicie uniezależnić logikę działania od interfejsu użytkownika. Taką możliwość daje nam VM, który nie ma żadnych bezpośrednich zależności z widokiem (a konkretniej jego implementacją). Zgodnie z wcześniejszą tezą jesteśmy w stanie uciąć powiązanie pomiędzy VM a pozostałymi warstwami i wykonać niezależne (automatyczne) testy logiki działania aplikacji. Wymaga to oczywiście przygotowania mockowanych modeli danych, jednak nie powinno to stanowić większego problemu. W ten sposób otrzymujemy trzy niezależne warstwy i każdą z nich jesteśmy w stanie osobno tworzyć i testować. Dodatkowo przy odpowiedniej konstrukcji warstwy te są niezależne od pozostałych komponentów aplikacji nie związanych bezpośrednio z MVVM, jak np. usług dostarczających dane i logikę biznesową czy konwerterów i stylów wspomagających konstrukcję UI.

### Problemy i rozwiązania

Podstawowym problemem przy separacji pomiędzy V a VM jest kwestia nawigacji w aplikacji oraz reakcji na działania użytkownika. W tym przypadku z pomocą przychodzą takie narzędzia jak Region Navigation, Behaviour no i oczywiście Delegate Command, dostępne w bibliotece Prism. Umożliwiają one (pewnym, acz wcale nie wielkim nakładem pracy) stworzenie warstwy pośredniczącej w komunikacji pomiędzy logiką a interfejsem, która będzie w wygodny i jednocześnie zgodny z MVVM sposób przekazywać informacje pomiędzy V a VM. W razie wątpliwości lub chęci poszerzenia swojej wiedzy polecam lekturę (dostępnej za darmo na stronie Prisma) książki _Developer's Guide to Microsoft Prism_. Dostarcza ona solidną dawkę informacji na temat konstrukcji aplikacji kompozytowych, zarówno jeżeli chodzi o UI, jak i strukturę całej aplikacji, a w dodatku napisana jest w dość przystępny i przejrzysty sposób. Ale oczywiście nic nie zastąpi praktyki, więc najlepsze co można zrobić aby dobrze zrozumieć wzorzec MVVM to usiąść i napisać choćby prostą aplikację, wykorzystując dostępną (lub zdobytą np. z podanej lektury) wiedzę. Powodzenia!