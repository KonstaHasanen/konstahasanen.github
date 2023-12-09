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
- Asiakaspalvelu

---

## Taidot

- **Ohjelmointikielet:** JavaScript
- **Frontend-kehitys:** HTML, CSS, Sveöte
- **Backend-kehitys:** Node.js, Express
- **Tietokannat:** MongoDB, MySQL
- **Pilvipalvelut:** AWS
- **Työkalut:** Git, VS Code

---

## Projektit

### [Ticorporate] - [2023/8] - [2023/12]
- **Sisältö:** Koulutusohjelmaan kuuluva noin 4kk mittainen sovelluskehitysprojekti, jossa tarkoituksena oli rakentaa toimiva sovelluskokonaisuus. Projekti toteutettiin viiden hengen ryhmissä Scrum projektinhallinta menetelmää hyödyntäen. Ryhmämme rakensi tapahtumasovelluksen, jossa vastasin backendistä, testauksesta sekä pilvipalveluista. 
- **Käytetyt tekniikat ja työkalut:**
  - Backend toteutettiin käyttämllä node.js, express, MongoDB stäckkiä.
  - Pilvipalveluinfastruktuuri rakennettiin AWS:n palveluita hyödyntäen, joita olivat S3, Elastic Beanstalk sekä CloudFront.
  - Sovelluskokonaisuutta testattiin Cypressillä.
- **Saavutukset ja oppimiskokemukset:**
  - S3 bucket käyttöönotto backendsovellukseen
  - Nodemailerin käyttö sähköpostin vahvistamiseen rekisteröinnin yhteydessä
  - Google autentikointi
  - Metodit tiedon manipuloimiseen
- **Koodi esimerkkejä oppimiskokemuksista:**
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
  


---

## Sertifikaatit

- [Sertifikaatin nimi] - [Suoritusvuosi]
- [Toinen sertifikaatin nimi] - [Suoritusvuosi]

---

## Kielitaito

- **Suomi:** Äidinkieli
- **Englanti:** Erinomainen
- **Ruotsi:** Perustaso

---
