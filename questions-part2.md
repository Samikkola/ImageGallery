# Kysymykset — Osa 2: Azure-julkaisu

Vastaa kysymyksiin omin sanoin. Lyhyet, selkeät vastaukset riittävät.

---

## Azure Blob Storage

**1.** Mitä eroa on `LocalStorageService.UploadAsync`:n ja `AzureBlobStorageService.UploadAsync`:n palauttamilla URL-arvoilla? Miksi ne eroavat?

> Localstorage URL (wwwroot/uploads...) no sovelluksen itsensä tarjoama polku ja käytettävissä vain sovelluksen kautta.
AzureBlobStorage URL (https://...blob.core.windows.net/...) on julkinen URL, joka osoittaa Azuren Blob Storage -palveluun.
Käytännössä BlobStorage on pilvessä, LocalSotrage lokaalissa.

---

**2.** `AzureBlobStorageService` luo `BlobServiceClient`:n käyttäen `DefaultAzureCredential()` eikä yhteysmerkkijonoa. Mitä etua tästä on? Mitä `DefaultAzureCredential` tekee eri ympäristöissä?

> DefaultAzureCredential mahdollistaa autentikoinnin "ketjutuksen", jolloin valitaan automattisesti oikea tunnistautumistapa, riippuen ympäristöstä.
Näin ei tarvitse kovakoodata yhteysmerkkijonoja, mikä nopeuttaa kehitystä ja parantaa turvallisuutta.

---

**3.** Blob Container luodaan `--public-access blob` -asetuksella. Mitä tämä tarkoittaa: mitä pystyy tekemään ilman tunnistautumista, ja mikä vaatii Managed Identityn?

> Käytännössä tämä asetus tarkoittaa sitä, että blobin sisältö on julkisesti luettavissa, mutta sisällön muokkaaminen vaatii tunnistautumisen Managed Identityn avulla.

---

## Application Settings

**4.** Application Settings ylikirjoittavat `appsettings.json`:n arvot. Selitä tämä mekanismi: miten se toimii ja miksi se on hyödyllistä eri ympäristöjä varten?

> Application Settings Azuress toimii Environment Variablejen tapaan, jolloin ne ovat prioriteetti järjestyksessä korkeammalla, ja niitä käytetään appsettingsin sijaan, mikäli ne ovat olemassa.
Tämä mahdollistaa ympäristökohtaiset asetukset ilman koodimuunnoksia.

---

**5.** Application Settingsissa käytetään `Storage__Provider` (kaksi alaviivaa), mutta koodissa luetaan `configuration["Storage:Provider"]` (kaksoispiste). Miksi?

>  Azuren Application Settings ei salli kaksoispisteitä avain-nimissä. Kaksi alaviivaa tunnistetaan ASP.NET Coressa kaksoispisteeksi.

---

**6.** Mitkä konfiguraatioarvot soveltuvat Application Settingsiin, ja mitkä eivät? Anna esimerkki kummastakin tässä tehtävässä.

> Applications Settingsiin sopivat arvot jotka eivät ole oikeasti salaisuuksia vaan pelkkiä konfiguraatio arvoja, esim. `Storage__Provider` tai `Storage__AccountName`.
Api-avaimet tai muut arvot joita ei haluta ulkopuolisten saataville eivät kuulu Applications Setttingsiin, esim.`ModerationService:ApiKet`.

---

## Managed Identity ja RBAC

**7.** Selitä omin sanoin: mitä tarkoittaa "System-assigned Managed Identity"? Mitä tapahtuu tälle identiteetille, jos App Service poistetaan?

> System-assigned Manadeg Identityssä App Servicelle luodaan identiteetti, jonka avulla se voi tunnistautua eei palveluihin, esim. Azure Blob Storageen.
Mikäli App Service poistetaan, myös idententiteetti poistuu automaattisesti. Tällöin vanhentuneita oikeuksia ei jää "roikkumaan".

---

**8.** App Servicelle annettiin `Storage Blob Data Contributor` -rooli Storage Accountin tasolle — ei koko subscriptionin tasolle. Miksi tämä on parempi tapa? Mikä periaate tähän liittyy?

> Contirubutor rooli antaa App Servicelle oikeuden lukea ja kirjoittaa vain tietyn accountin sisällä olevia Blobeja, ei muita.
Näin rajoitetaan pääsyä vain niihin resursseihin, joihin App Servicen oikeasti tarvitsee päästä.
Tämä noudattaa `Least Privilege` -peritaatetta, jossa käyttäjälle/palvelulle annetaan vain ne pikeudet jotka se tarvitsee, eikä yhtään enemäpää.

---


