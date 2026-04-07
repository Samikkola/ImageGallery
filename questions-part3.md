# Kysymykset — Osa 3: Key Vault ja Infrastructure as Code

Vastaa kysymyksiin omin sanoin. Lyhyet, selkeät vastaukset riittävät.

---

## Key Vault

**1.** Miksi `ModerationService:ApiKey` tallennettiin Key Vaultiin eikä Application Settingsiin? Mitä lisäarvoa Key Vault tuo Application Settingsiin verrattuna?

> KeyVault turvallinen paikka säilöä salaisuuksia, sillä se on Azuren tarjoama 'holvi' johon tallennetut asiat eivät näy selkokielisesti ulkopuolisille. Se tarjoaa salauksen lepotilassa, versiohistorian, auditoinnin, pääsynhallinan sekä rotaation. Applications Settingisiin verrattuna, se salaa tiedot turvallista säilytystä varten.

---

**2.** Key Vault -salaisuuden nimi on `ModerationService--ApiKey` (kaksi väliviivaa), mutta koodissa se luetaan `configuration["ModerationService:ApiKey"]` (kaksoispiste). Miksi käytetään `--`?

> key Vault ei hyväksy kaksoispisteitä salaisuuksien nimissä, vaan käyttää kahta väliviivaa. Nämä muunnetaan automaattisesti konfiguraatiohierarkisesti (-- -> :) tarvittaessa.

---

**3.** `Program.cs`:ssä Key Vault lisätään konfiguraatiolähteeksi `if (!string.IsNullOrEmpty(keyVaultUrl))`-ehdolla. Miksi tämä ehto on tärkeä? Mitä tapahtuisi ilman sitä?

> Tämä ehto mahdollistaa ympäristöjen ajamisen samalla koodilla. Kun keyVaultUrl on tyhjä, eli Azuren Applications Settingsissä ei ole mitään, kehitetään lokaalisti. Kun arvo taas löytyy, yhditetään Keu Vaultiin automaattisesti. Jos ehtoa ei olisi, yrittäisi sovellus aina yhdistää Key Vaultiin, mikä todennäköisesti aiheuttaa virheitä lokaalissa kehityksessä.

---

**4.** Kun sovellus on käynnissä Azuressa, konfiguraation prioriteettijärjestys on: Key Vault → Application Settings → `appsettings.json`. Selitä millä arvolla `ModerationService:ApiKey` lopulta ladataan — ja käy läpi jokainen askel siitä, miten arvo päätyy sovelluksen `IOptions<ModerationServiceOptions>`:iin.

> Sovelluksen käynnistyessä se lataa konfiguraatio arvon ensi `appsettings.json`:nista, sen jälkeen siirtyy tarkastamaan Applications Settingsin. Jos sieltä löytyy sama konfiguraatio, ylikirjoitetaan edellinen. Tätä jatketaan kunnes päädytään ylimmän prioriteetin konfiguraatio arvoon, eli tässä tapauksessa se saadaan Key Vault secretinä. ``builder.Configuration.AddAzureKeyVault(new Uri(keyVaultUrl), new DefaultAzureCredential());` -koodi lisää Key Vaultin konfiguraatiolähteeksi, jonka jälkeen ModerationServicen avulla apikey saadaan lisättyä sovellukseen.

---

**5.** Mitä eroa on `Key Vault Secrets User` ja `Key Vault Secrets Officer` -roolien välillä? Miksi annettiin nimenomaan `Secrets User`?

> `User` -roolilla saadaan vain lukea ja listata secretit, kun taas `Officer` -rooli saa mm. luoda ja päivittää secretejä. `User` -roolin käyttö perustuu jälleen `Least privilege` -periaatteeseen, jossa resurssille annetaan vain ne oikeudet joita se aidosti tarvitsee.

---

## Infrastructure as Code (Bicep)

**6.** Bicep-templatessa RBAC-roolimääritykset tehdään suoraan (`storageBlobRole`, `keyVaultSecretsRole`). Mitä etua tällä on verrattuna siihen, että ajat erilliset `az role assignment create` -komennot käsin?

> Kun myös roolimääritykset liitetään osaksi Bicep-templaattia, ovat ne mukana IaC-määrittelyssä. Tämä tarjoaa useita hytyjä manuaalisiin komentoihin verrattuna, kuten toistettavuuden, idempotenttisuuden, versionhallinnan ja deployaksen täyden automatisoinnin. 

---

**7.** Bicep-parametritiedostossa `main.bicepparam` on `param moderationApiKey = ''` — arvo jätetään tyhjäksi. Miksi? Miten oikea arvo annetaan?

> Oikea arvo annetaan deplyo-komennossa, sillä `main.bicepparam` -tiedosto on osa gitin trackaamiä tiedostoja, jolloin kaikki sen sisältämä menee githubiin. 

---

**8.** Bicep-templatessa `webApp`-resurssin `identity`-lohkossa on `type: 'SystemAssigned'`. Mitä tämä tekee, ja mitä manuaalista komentoa se korvaa?

> Tämä asettaa sovellukselle automaattisesti oman managed identityn Azuressa, jota voidaan käyttää mm. autentikointiin ilman salasanoja. Manuaalinen komento tälle olisi `az webapp identity assign....`

---

**9.** RBAC-roolimäärityksen nimi generoidaan `guid()`-funktiolla:

```bicep
name: guid(storageAccount.id, webApp.identity.principalId, 'StorageBlobDataContributor')
```

Miksi nimi generoidaan näin eikä esimerkiksi kovakoodatulla merkkijonolla? Mitä tapahtuisi jos nimi olisi sama kaikissa deploymenteissa?

> RBAC-roolimäärityksen nimi täytyy olla globaalisti uniikki sekä GUID-arvo, jolloin funktion käyttö on helpoin tapa saavuttaa nämä vaatimukset. Käyttämälle resurssien arvoja syötteinä, varmistetaan että sama resurssi saa aina saman tunnisteen, jolloin deployment pysyy idempotenttina, eikä se voi epäonnistua nimien konfliktien takia.

---

**10.** Olet nyt rakentanut saman infrastruktuurin kahdella tavalla: manuaalisesti (Osat 2 & 3) ja Bicepillä (Osa 3). Kuvaile konkreettisesti yksi tilanne, jossa IaC-lähestymistapa on selvästi manuaalista parempi. Kuvaile myös tilanne, jossa manuaalinen tapa riittää.

> Kun työskennellään useissa eri ympäristöissä, usean kehittäjän voimin, tulee IaC-lähstymistavan hyödyt parhaiten esiin. Näin dokumentaatio on aina ajantasainen (koodi ja git-historia), ympäristöjen pystytys on automatisoinnin ansiosta nopeaa ja vaihtaminen vaivatonta. Mikäli kehitystyö on vielä varhaisessa vaiheessa tai on tarkoitus vain testata eri skenaarioita, manuaalinen tapa riittää mainiosti.
