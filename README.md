# Tietojenkäsittely

---

## Koulutus

### Lukio [2019] - [2021]
**Schildtin lukio**

### Tietojenkäsittely Tradenomi [2021] - [2024]
**Jyväskylän ammattikorkeakoulu**

---

## Työkokemus

### [Sakupe Oy] - [2020] - [2024]
**Tehtävä:**
- Asiakaspalvelutehtävät Sairaala Novan vaateautomaatilla

---

## Taidot

- **Ohjelmointikielet:** JavaScript
- **Frontend-kehitys:** HTML, CSS, Svelte
- **Backend-kehitys:** Node.js, Express
- **Tietokannat:** MongoDB, MySQL
- **Pilvipalvelut:** AWS
- **Työkalut:** Git, VS Code

---

## Projektit

### [Ticorporate] - [2023/8] - [2023/12]
- **Sisältö:**
  - Koulutusohjelmaan kuuluva noin 4kk mittainen sovelluskehitysprojekti, jossa tarkoituksena oli rakentaa toimiva sovelluskokonaisuus. Projekti toteutettiin viiden hengen ryhmissä Scrum projektinhallinta menetelmää hyödyntäen. Ryhmämme rakensi tapahtumasovelluksen, jossa vastasin backendistä, testauksesta sekä pilvipalveluista. 
- **Käytetyt tekniikat ja työkalut:**
  - Backend toteutettiin käyttämllä node.js, express, MongoDB stäckkiä.
  - Pilvipalveluinfastruktuuri rakennettiin AWS:n palveluita hyödyntäen, joita olivat S3, Elastic Beanstalk sekä CloudFront.
  - Sovelluskokonaisuutta testattiin Cypressillä.
- **Saavutukset ja oppimiskokemukset:**
  - S3 bucket käyttöönotto backendsovellukseen
  - Nodemailerin käyttö sähköpostin vahvistamiseen rekisteröinnin yhteydessä
  - Google autentikointi
  - Metodit tiedon manipuloimiseen
  - Manuaalitestaustapahuman suunnittelu, seuraaminen sekä tulosten raportointi
- **Koodi esimerkkejä oppimiskokemuksista:**
  - S3 bucketin käyttöönotto, jotta voidaan tallentaa kuvia tapahtuman lisäämisen yhteydessä. Toiminnallisuuden totetuttaminen vaati S3 bucketin konfiguraatiota AWS:n puolella sekä backendsovelluksessa. Backendsovellukseen luotiin awsConfig.js niminen tiedosto, jossa tehtiin uusi S3Client, jota voidaan myöhemmin käyttää tapahtuman lisäämismetodissa.

  - awsConfig.js tiedoston sisältö

`````
const { S3Client } = require('@aws-sdk/client-s3');
require('dotenv').config();

const bucketRegion = process.env.BUCKET_REGION;
const accessKey = process.env.ACCESS_KEY;
const secretAccessKey = process.env.SECRET_ACCESS_KEY;

const s3 = new S3Client({
  credentials: {
    accessKeyId: accessKey,
    secretAccessKey: secretAccessKey,
  },
  region: bucketRegion,
});

module.exports = {
  s3,
};

`````
  - Tapahtuman lisäämiseen käytettävä addEvent metodi, jossa käytössä S3Client

`````
 addEvent: async (req, res) => {
    try {
      if (req.file) {
        const imageFile = req.file;

        const uploadParams = {
          Bucket: bucketName,
          Key: imageFile.originalname,
          Body: imageFile.buffer,
          ContentEncoding: 'base64',
          ContentDisposition: 'inline',
          ContentType: 'image/jpeg',
        };

        const uploadCommand = new PutObjectCommand(uploadParams);
        await s3.send(uploadCommand);

        const objectUrl = `https://${bucketName}.s3.amazonaws.com/${imageFile.originalname}`;

        // Luodaan uusi tapahtuma
        const newEvent = new Event({
          nimi: req.body.nimi,
          kuvaus: req.body.kuvaus,
          tapahtumapaikka: req.body.tapahtumapaikka,
          genre: req.body.genre,
          aloitusaika: req.body.aloitusaika,
          aloituspvm: req.body.aloituspvm,
          lopetusaika: req.body.lopetusaika,
          lopetuspvm: req.body.lopetuspvm,
          sijainti: req.body.sijainti,
          kuvaUrl: objectUrl,
        });

        try {
          //Tallennetaan tapahtuma events collectioniin
          await newEvent.save();

          //Lisätään tapahtuma users events alidokumenttiin, jotta tiedetään myöhemmin kuka on lisännyt tapahtuman
          const user = await User.findByIdAndUpdate(
            req.params.id,
            {
              $push: { events: newEvent },
            },
            { new: true } // Palautetaan päivitetty käyttäjä
          );

          // response jos lisääminen onnistuu
          res.json({ event: newEvent, user });
        } catch (error) {
          //Virheenkäsittely, jos lisääminen ei onnistu
          console.error('Error saving event or updating user:', error);
          res
            .status(500)
            .json({ error: 'Error saving event or updating user' });
        }
      } else {
        res.status(400).json({ error: 'No file provided' });
      }
    } catch (error) {
      console.error('Error adding new event:', error);
      res.status(500).json({ error: 'Internal server error' });
    }
  },

`````

  - Nodemailerin käyttöönotto rekisteröinnin yhteydessä. Tarkoituksena, että käyttäjä vahvistaa käyttämänsä sähköpostiosoitteen klikkaamalla sähköpostiin tulevaa linkkiä rekisteröitymisen jälkeen, jolloin voi vasta käyttää tunnuksia kirjautumiseen. Toiminnallisuus on toteutettu käyttämällä MAP-tietorakennetta, jolloin käyttäjän tiedot varastoidaan väliaikaisesti siihen asti, kunnes sähköposti on vahvistettu.

`````
// Nodemailerin konfiguraatio
const transporter = nodemailer.createTransport({
  host: 'smtp.gmail.com',
  port: 587,
  secure: true,
  service: 'Gmail',
  auth: {
    user: 'konsta.hasanen@gmail.com',
    pass: 'svnw bxni avsi zcvu',
  },
});

/*
Käytetään Map tietorakennetta tässä, jotta voidaan säilyttää käyttäjän tietoja väliaikaisesti, kunnes sposti on vahvistettu onnistuneesti
*/

const temporaryUserMap = new Map();

const UserController = {
  //Uuden käyttäjän rekisteröinti ja sposti varmennus nodemailerin avulla
  registerNewUser: async function (req, res, next) {
    try {
      //Tarkistetaan aluksi onko syötetty sposti jo käytössä
      const existingUser = await User.findOne({ sposti: req.body.sposti });
      if (existingUser) {
        console.error('User registration failed: Sähköposti on jo käytössä.');
        return res
          .status(500)
          .json({ success: false, message: 'Sähköposti on jo käytössä.' });
      }

      //Salasanan hashaus
      const hashedPassword = bcrypt.hashSync(req.body.salasana, 8);

      //Käyttäjä objekti, jossa vaadittavat tiedot
      const user = {
        etunimi: req.body.etunimi,
        sukunimi: req.body.sukunimi,
        sposti: req.body.sposti,
        salasana: hashedPassword,
        emailVerified: false,
        confirmationCode: crypto.randomBytes(16).toString('hex'),
      };

      // Tallenna käyttäjä väliaikaiseen tietorakenteeseen (Map)
      temporaryUserMap.set(user.confirmationCode, user);

      const confirmationLink = `http://finde.us-east-1.elasticbeanstalk.com/users/confirm/${user.confirmationCode}`;
      //Vahvistusviestin sisältö mailOptions objektissa
      const mailOptions = {
        from: 'finde@gmail.com',
        to: user.sposti,
        subject: 'Vahvista sähköpostiosoitteesi',
        html: `Tervetuloa käyttämään FindE sovellusta, klikkaa <a href="${confirmationLink}">tästä</a> vahvistaaksesi sähköpostiosoitteesi.`,
      };

      transporter.sendMail(mailOptions, function (error, info) {
        if (error) {
          console.error('Error sending confirmation email:', error);
        } else {
          console.log('Confirmation email sent:', info.response);
        }
      });

      res.json({
        success: true,
        message: 'Tarkista sähköpostisi vahvistaaksesi tilin.',
      });
    } catch (error) {
      console.error('User registration failed:', error);
      res.status(500).send('User registration failed.');
    }
  },

`````
  - Google autentikointi sovellukseen

`````  
/*
Tokenin validointi google-auth-library -kirjaston avulla. Toiminta esitelty
osoitteessa: https://developers.google.com/identity/sign-in/web/backend-auth
Tässä siis validoidaan frontendistä saatu Googlen token eli varmistetaan että
oikea käyttäjä pääsee käyttämään backendiä.
*/
const { OAuth2Client } = require('google-auth-library');
const GOOGLE_CLIENT_ID = process.env.GOOGLE_CLIENT_ID; // Googlen clientId haetaan .env-filusta

const client = new OAuth2Client(GOOGLE_CLIENT_ID);

/* Frontendistä saadun Googlen idTokenin validointi Googlen palvelussa
   ES2017:ssa esitelty async -funktio tekee tästä paljon mukavamman
   näköisen verrattuna siihen jos olisi tehty callbackilla
*/
async function validateSocialToken(token) {
  const ticket = await client.verifyIdToken({
    idToken: token,
    audience: GOOGLE_CLIENT_ID,
  });
  const payload = ticket.getPayload();
  const gmail = payload['email'];
  console.log('saatiin sposti ' + gmail);
  const userid = payload['sub'];
  console.log('Saatiin userid: ' + userid);
  return gmail; // gmail palautetaan promisena, jotta voidaan käyttää sitä myöhemmmin
}

module.exports = validateSocialToken;

`````
  - Autentikoinnin käyttö sisäänkirjautumisessa, authenticateGUser metodilla

`````
authenticateGUser: async function (req, res, next) {
    try {
      // Googlen tokeni frontendistä
      const token = req.body.gtoken;

      // Varmistetaan, että saatu token on oikea
      const gmail = await validateSocialToken(token);

      // Tarkistetaan, onko sähköpostilla jo käyttäjä kannassa
      const existingUser = await User.findOne({ sposti: gmail });

      if (existingUser) {
        // Jos käyttäjä löytyy luodaan JWT token ja lähetetään se frontendiin
        const user = existingUser;
        const jwttoken = createToken(user);
        res.json({
          success: true,
          message: 'Google user authenticated successfully',
          token: jwttoken,
        });
      }
      // Jos käyttäjää ei löydy, luodaan uusi newUser objekti seuraavilla tiedoilla
      else {
        const newUser = {
          etunimi: 'Google',
          sukunimi: 'User',
          sposti: gmail,
          salasana: 'googlepassword',
          emailVerified: true,
        };

        // Tallennetaan käyttäjä tietokantaan
        const createdUser = await User.create(newUser);

        // Luodaan käyttäjälle JWT token ja lähetetään se frontendiin
        const user = createdUser;
        const jwttoken = createToken(user);
        res.json({
          success: true,
          message: 'Google user authenticated successfully',
          token: jwttoken,
        });
      }
    } catch (error) {
      // Virheenkäsittely, jos autenktikointi jostain syystä epäonnistuu
      console.error('Google authentication failed:', error);
      res.status(500).send('Google authentication failed.');
    }
  },
`````
  - Järjestimme manuaalitestaustapahtuman mobiilisovelluksen käytettävyyden testaamista varten. Suunnittelin ennen testaustapahtumaa taskeja, joita testihenkilön tulisi suorittaa testaustapahtumassa. Testaustapahtuman
    edetessä kirjasin ylös testihenkilön havaintoja mobiilisovelluksen käytetävyyteen liityen, jotta voisimme parantaa käytettävyyttä. Lopuksi tein tapahtumasta vielä raportin, jossa huomiot nousi esiin, jonka toimitin
    mobiilisovelluksen kehityksestä vastaavalle henkilölle.
