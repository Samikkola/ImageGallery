# Kysymykset — Osa 1: Lokaali kehitys

Vastaa kysymyksiin omin sanoin. Lyhyet, selkeät vastaukset riittävät — tarkoitus on osoittaa, että olet ymmärtänyt konseptit.

---

## Clean Architecture

**1.** Selitä omin sanoin: mitä tarkoittaa, että `UploadPhotoUseCase` "ei tiedä" tallennetaanko kuva paikalliselle levylle vai Azureen? Näytä koodirivit, jotka osoittavat tämän.

> UseCase käyttää rajapintaa, joka määrittää vain tarvittavat metodit, mutta ei kerro miten ne toteutetaan. 
Näin UseCase on riippumaton tallennustavasta, joka määritellään appsettings tiedoston arvojen avulla Infrastructure-kerroksen DepencyInjectionissa:
```
var provider = configuration[$"{StorageOptions.SectionName}:Provider"]
            ?? StorageOptions.LocalProvider;
        if (provider == StorageOptions.AzureProvider)
            services.AddScoped<IStorageService, AzureBlobStorageService>();
        else
            services.AddScoped<IStorageService, LocalStorageService>();
```
---

**2.** Miksi `IStorageService`-rajapinta on määritelty `GalleryApi.Domain`-kerroksessa, mutta `LocalStorageService` on `GalleryApi.Infrastructure`-kerroksessa? Mitä hyötyä tästä jaosta on?

> Näin Domain-kerros määrittelee vain tarvittavat rajapinnat, eikä sisällä toteutuksia. 
Tämä mahdollistaa sen, että voidaan helposti vaihtaa toteutusta ilman, että Domain-kerros tarvitsee tietää siitä.

---

**3.** Testit käyttävät `Mock<IAlbumRepository>`. Mitä mock-objekti tarkoittaa, ja miksi Clean Architecture tekee tämän testaustavan mahdolliseksi?

> Mock-objekti on testeissä käytetty "korvike" oikealle objektille, joka toteuttaa rajapinnan.
Näin voidaan testata UseCasea ilman, että tarvitaan oikeaa tietokantaa.
Tämä on mahdollista Clean Architecturen ansiosta, koska UseCase on riippumaton toteutuksista ja käyttää vain rajapintoja.

---

## Salaisuuksien hallinta

**4.** Kovakoodattu API-avain on ongelma, vaikka repositorio olisi yksityinen. Selitä kaksi eri syytä miksi.

> Vaikka repo olisi yksityinen, voi siihen olla pääsy usealla eri henkilöllä, ja avain voi vuotaa vahingossa esim. jos reposta tehdään julkinen tai jos joku jakaa koodia eteenpäin. 
Commit-historiaan jää myös jälki avaimesta, jolloin se on löydettävissä vaikka sen poistaisikin myöhemmin.

---

**5.** Riittääkö se, että poistat kovakoodatun avaimen myöhemmässä commitissa? Perustele vastauksesi.

> Ei. Commitit jäävät elämään Git-historiaan, joten avain on edelleen löydettävissä vanhoista committeista.

---

**6.** Minne User Secrets tallennetaan käyttöjärjestelmässä? (Mainitse sekä Windows- että Linux/macOS-polut.) Miksi tämä sijainti on turvallinen?

> Windows: %APPDATA%\Microsoft\UserSecrets\<user_secrets_id>\secrets.json
Linux/macOs: ~/.microsoft/usersecrets/<user_secrets_id>/secrets.json
Nämä sijainnit eivät ole osa projektia, joten ne eivät päädy versionhallintaan.
Lisäksi ne ovat käyttäjäkohtiasia, joten kenelläkään muulla ei pitäis olla pääsyä niihin.

---

## Options Pattern ja konfiguraatio

**7.** Mitä hyötyä on `IOptions<ModerationServiceOptions>`:n käyttämisestä verrattuna siihen, että luetaan arvo suoraan `IConfiguration`-rajapinnalta (`configuration["ModerationService:ApiKey"]`)?

> IOptions tarjoaa tyypityksen, tuo tuen intellisenselle ja helpottaa konfiguraation hallintaa ja dokumentointia, sillä ne voidaan pitää yhdessä paikassa.

---

**8.** ASP.NET Core lukee konfiguraation useista lähteistä prioriteettijärjestyksessä. Listaa lähteet korkeimmasta matalimpaan ja selitä, mikä arvo lopulta käytetään, kun sama avain on sekä `appsettings.json`:ssa että User Secretsissä.

> Konfiguraatiolähteet prioriteettijärjestyksessä korkeimmasta matalimpaan: 1) Environment Variables 2) User Secrets 3) appsettings.develompent.json 4) appsettings.json
Näin ollen saman avaimen ollessa sekä appsettings.json:ssa että User Sercertissä, User Secret luetaan jälkimmäisenä ja sitä käytetään.

---

**9.** `DependencyInjection.cs`:ssä valitaan tallennustoteutus näin:

```csharp
var provider = configuration["Storage:Provider"] ?? "local";
if (provider == "azure")
    services.AddScoped<IStorageService, AzureBlobStorageService>();
else
    services.AddScoped<IStorageService, LocalStorageService>();
```

Miksi käytetään konfiguraatioarvoa `env.IsDevelopment()`-tarkistuksen sijaan? Mitä haittaa olisi `if (env.IsDevelopment()) { käytä lokaalia }`-lähestymistavassa?

> Mikäli käytetään `env.IsDevelopment()`-tarkistusta, muuttuu sovellus paljon jäykemmäksi ja toteutuksen muokkaaminen vaikeammaksi.
Kun käytetään konfiguraatioarvoa, voidaan eri toteutusten välillä vaihtaa helpoommin, ilman suuria koodi muunnoksia.


---

## Tiedostotallennus

**10.** Kun lataat kuvan, `imageUrl`-kentän arvo on `/uploads/abc123-..../photo.jpg`. Miten tähän URL:iin pääsee selaimella? Mihin koodiin tämä perustuu?

> Tähän urliin pääsee selaimella suoraan tämän program.cs tiedostossa määritellyn middlewaren ansiosta
```
app.UseStaticFiles();
```
Tämä mahdollistaa staattisten tiedostojen tarjoamisen wwwroot-kansiosta.

---

**11.** Mitä tapahtuu jos yrität ladata tiedoston jonka MIME-tyyppi on `application/pdf`? Missä tiedostossa ja millä koodirivillä tämä käyttäytyminen on määritelty?

> Sovellus palauttaa HTTP -virheen, koska UploadPhotoUseCase tarkstaa tiedoston tyypin seuraavilla koodiriveillä:
.Application\UseCases\Photos\UploadPhotoUseCase.cs - rivit 14-16,
määritellään sallitut kuvatyypit:

```
// Sallitut kuvatyypit
    private static readonly string[] AllowedContentTypes =
        ["image/jpeg", "image/png", "image/webp", "image/gif"];
```
.Application\UseCases\Photos\UploadPhotoUseCase.cs - rivit 41-44,
tarkastaa tiedostotyypin ja palauttaa mahdollisen virheen:
```
// 2. Validoi tiedostotyyppi
        if (!AllowedContentTypes.Contains(request.ContentType))
            return Result<PhotoDto>.Failure(
                $"Tiedostotyyppi '{request.ContentType}' ei ole sallittu. " +
                $"Sallitut tyypit: {string.Join(", ", AllowedContentTypes)}");
```


---

**12.** `DeletePhotoUseCase` poistaa tiedoston kutsumalla `_storageService.DeleteAsync(photo.FileName, photo.AlbumId)` — ei `photo.ImageUrl`:lla. Miksi?

> Absoluuttinen polku on turvallinen tapa poistaa tiedosto sillä se osoittaa varmasti oikeaan paikkaan, photo.ImageUrl on vain julkinen oisoite josta kuva löytyy.
Myöskään azuren polku ei välttämättä vastaa suoraan url:ia.