# Detekcija bot profila na društvenoj mreži Twitter primenom neuronskih mreža

## 1. Opis problema

Savremeni informacioni prostor, posebno na društvenim mrežama, karakteriše izuzetno brz protok informacija, ali i sve veća prisutnost dezinformacija i manipulativnog sadržaja. Automatizovani nalozi (botovi) predstavljaju jedan od ključnih izazova u analizi komunikacije na društvenim mrežama, jer mogu da naruše prirodne tokove konverzacije, šire dezinformacije i simuliraju lažnu društvenu podršku.

Cilj ovog projekta jeste razvoj modela za automatsku detekciju bot profila na društvenoj mreži Twitter primenom višeslojne feedforward neuronske mreže. Model se trenira na označenom skupu podataka koji sadrži poznate bot i human profile, a zatim se primenjuje za klasifikaciju novih, neoznačenih profila iz novog skupa podataka koji obuhvata diskusije o COVID-19 pandemiji.

## 2. Podaci

### Trening podaci
Skup podataka korišćen u ovom radu preuzet je sa javne platforme Kaggle i obuhvata podatke prikupljene sa društvene mreže Twitter. Podaci su prikupljeni i prethodno obrađeni od strane autora dataset-a, a dataset dostupan je na sledećem linku: https://www.kaggle.com/datasets/danieltreiman/twitter-human-bots-dataset.
Dataset sadrži 37,438 Twitter profila, od čega je 66.8% human profila, a 33.3% automatizovanih (bot) profila. Dataset sadrži 23 kolone, od kojih su za ovaj rad najznačajnije sledeće:

| Kolona | Opis |
|---|---|
| `followers_count` | Broj pratilaca |
| `friends_count` | Broj profila koje korisnik prati |
| `favourites_count` | Broj lajkovanih tvitova |
| `statuses_count` | Ukupan broj objavljenih tvitova |
| `average_tweets_per_day` | Prosečan broj tvitova po danu |
| `account_age_days` | Starost naloga u danima |
| `description` | Opis profila |
| `location` | Lokacija korisnika |
| `account_type` | Labela: `bot` ili `human` |

### Podaci za predviđanje
Skup podataka korišćen za predikciju je takođe preuzet sa platforme Kaggle i nalazi se na sledećem linku: https://www.kaggle.com/datasets/abhishek252/covid19-tweets-dataset. Dataset obuhvata tvitove prikupljene u periodu mart–maj 2020. godine, tokom prve faze karantina u Indiji. Za predikciju korišćeno je jezgro mreže dobijeno k-core dekompozicijom (k=5), koje sadrži 600 korisnika.

### Analiza i obrada
Nebalansiranost klasa rešena je primenom tehnike SMOTE (Synthetic Minority Oversampling Technique) isključivo na trening skupu:
Pre SMOTE:  human = 17519, bot = 8702
Posle SMOTE: human = 17519, bot = 17519

Feature engineering: izvučeno je 9 feature-a po profilu. Određene informacije su uzete direktno iz kolona, koje su prethodne objašnjene, dok su neki ('description_length', 'has_description', 'has_location') napravljeni na osnovu postojećih kolona.

## 3. Arhitektura modela
Korišćena je višeslojna feedforward neuronska mreža implementirana u biblioteci PyTorch. Ulazni sloj prima 9 feature-a, koje dalje prosleđuje kroz tri skrivena sloja koji imaju 64, 32 i 16 neurona respektivno. Na svim slojevima primenjena je aktivaciona funkcija ReLU koja unosi nelinearnost u model, kao i Dropout koji gasi 30% nasumičnih neurona što omogućava da ne dođe do pretreniranosti mreže. Na poslednjem, izlaznom sloju koji ima samo jedan neuron, primenjena je Sigmoidna funkcija, s obzirom da je u pitanju binarna klasifikacija, što omogućava interpretaciju izlaza kao verovatnoće da je profil bot (vrednost između 0 i 1).

## 4. Trening
Podaci su podeljeni u trening, test i validacioni skup i to 70% podataka pripada trening skupu, dok po 15% pripada test i validacionom skupu. Pre treniranja primenjena je StandardScaler normalizacija na trening skupu (`fit_transform`), dok je na test i prediction skupu primenjena samo transformacija (`transform`) koristeći parametre naučene iz trening skupa. Za treniranje korišćen je Adam algoritam optimizacije, Binary Cross Entropy standardna loss funkcija za binarnu klasifikaciju. Korišćena je tehnika Early Stopping kako ne bi došlo do pretreniranost mreže i ona je zaustavila treniranje u 61. epohi.

## 5. Analiza osetljivosti i hiperparametarska optimizacija
U okviru projekta analiziran je uticaj različitih ulaznih atributa na performanse modela. Prvo je model testiran sa 9 ulaznih parametara kada je ostvario tačnost od 81%, nakon čega su dva atributa koja nisu eksplicitno postojala u datasetu za predikciju izbačena čime je model ostvario tačnost od 77%. Takođe parametri za Dropout koji su testirani su 0.2 i 0.3 pri čemu bolji rezultat donosi vrednost od 30%. Takođe, uvođenjem tehnike Early Stopping može se primetiti da je veoma slična tačnost modela (81%), kao i kada se eksplicitno definiše broj epoha (50 epoha) i kada je dataset podeljen isključivo na trening i test skup (80%).

## 6. Rezultati evaluacije
Performanse modela evaluirane su na test skupu korišćenjem metrika Accuracy, Precision, Recall i F1-score. Razvijena su i upoređena dva modela koji se razlikuju po metodologiji podele podataka, međutim tačnost oba modela je veoma slična i iznosi 81% i 80% dok su razlike u ostalim metrikama male, ali ipak su kod drugog modela bolje. Vrednosti za model gde je korišćena tehnika Early Stopping su sledeće:
        precision    recall  f1-score   

Human   0.90         0.81      0.85      
Bot     0.68         0.81      0.74          
Može se primetiti da je preciznost modela u predikciji human profila 90%, što znači da od svih profila koje je klasifikovao kao human profile 90% tih profila su zaista human, dok kod klase bot preciznost je malo niža, a to je posledica što sam dataset na početku nije bio izbalansiran, a veći deo skupa su činili profili upravo human klase, koje je sam model bolje naučio da prepoznaje.

## 7. Diskusija

Nakon što je model postigao odgovarajuću tačnost, primenjen je na novom skupu podataka gde je bilo neophodno identifikovati botove. Skup podataka je bio ograničen na 600 korisnika, koji pripadaju jezgru ranije identifikovane mreže (mreža twitter profila je nastala za potrebe projektnog rada iz predmeta Napredna analiza podataka). Cilj je bio potvrditi pretpostavku da pomenuti skup podataka pokazuje karakteristike organske javne rasprave. Nakon primene binarne klasifikacije, neuronska mreža identifikovala je 16% (98 profila od 600) automatizovanih profila, što ukazuje da jezgro mreže nema dovoljan broj automatizovanih profila koji bi mogli da utiču na tok diskusije. Dodatno, profil `@ImRaina`, koji je u SNA analizi identifikovan kao organska javna ličnost (popularni indijski sportista), dobio je izrazito nisku verovatnoću bot klasifikacije (3.7%), što potvrđuje konzistentnost između dve primenjene metode.

Takođe, postoje i određena ograničenja. Naime, model je treniran na podacima iz 2019. godine, dok prediction dataset potiče sa početka 2020. Botovi koji su delovali tokom COVID-19 pandemije mogli su imati drugačije obrasce ponašanja od botova u trening datasetu. Zatim, kolone `average_tweets_per_day` i `account_age_days` nisu bile dostupne u prediction datasetu i ručno su izračunate na osnovu drugih podataka koji su bili dostupni, što može uticati na kvalitet predikcije. Dodatno, model klasifikuje profile isključivo na osnovu metapodataka profila, bez analize tekstualnog sadržaja tvitova. Uključivanje NLP feature-a (sentimentalna analiza) moglo bi dodatno poboljšati performanse.

## 8. Zaključak

U ovom projektu razvijena je višeslojna feedforward neuronska mreža za detekciju bot profila na društvenoj mreži Twitter. Model postiže tačnost od 80% na test skupu, što predstavlja solidne rezultate za zadatak binarne klasifikacije.

Primenom modela na jezgro Twitter mreže formiranog od diskusija o COVID-19 pandemiji, utvrđeno je da svega 16% profila pokazuje karakteristike automatizovanih naloga. Ovaj nalaz je konzistentan sa rezultatima prethodno sprovedene analize društvenih mreža, koja je takođe ukazala na dominantno organsku prirodu posmatrane komunikacije.

Budući pravci istraživanja uključuju proširenje skupa feature-a tekstualnim i vremenskim karakteristikama tvitova, kao i primenu naprednijih arhitektura poput BERT modela za direktnu analizu sadržaja objava.








