# Koha-Suomi plugin SIPoHTTP
Adds SIPoHTTP support for Koha
# Downloading
From the release page you can download the latest \*.kpz file
# Installing
Koha's Plugin System allows for you to add additional tools and reports to Koha that are specific to your library. Plugins are installed by uploading KPZ ( Koha Plugin Zip ) packages. A KPZ file is just a zip file containing the perl files, template files, and any other files necessary to make the plugin work.
The plugin system needs to be turned on by a system administrator.
To set up the Koha plugin system you must first make some changes to your install.
    Change <enable_plugins>0<enable_plugins> to <enable_plugins>1</enable_plugins> in your koha-conf.xml file
    Confirm that the path to <pluginsdir> exists, is correct, and is writable by the web server
    Remember to allow access to plugin directory from Apache
    <Directory <pluginsdir>>
        Options Indexes FollowSymLinks
        AllowOverride None
        Require all granted
    </Directory>
    Restart your webserver
Once set up is complete you will need to alter your UseKohaPlugins system preference. On the Tools page you will see the Tools Plugins and on the Reports page you will see the Reports Plugins.
# Konfigurointi
Laitteet, jotka käyttävät sipohttp:tä tarvitsevat sip2-palvelimen konfiguraatiotiedostoon (esim. SIPConfig.xml) <login></login>-tunnustensa yhteyteen (<accounts></accounts> sisällä) määrityksen sipohttp="XXXXXXX".
Tämän lisäksi konfiguraatiotiedostoon on lisättävä nimi, ip- ja porttimääritys sipohttp-parametrin määrittämiseksi (<sipohttp></sipohttp>). Näillä määrityksillä SIP2-viestiliikenne ohjataan haluttuun sip2-palvelinosoitteeseen. SIP2-palvelimen konfiguraatiotiedoston transport-tyypiksi on asetettava "RAW". Timeout- eli aikakatkaisuarvon oletus on 5 sekuntia, jonka jälkeen mahdollinen edelleen avoin yhteys rajapinnan ja SIP2-palvelimen välillä katkaistaan. Jos aikakatkaisu tapahtuu, lähettää rajapinta paluuviestinä XML-viestin, jossa on tyhjä SIP2-viesti response-osassa.

Esimerkki määritystiedostosta SIPconfig.xml, jossa sipohttp-liikenne ohjataan konfiguraatiotiedoston tiedoilla käynnistettyyn samaiseen sip2-palvelimeen (127.0.0.1:6009):

    <acsconfig xmlns="http://openncip.org/acs-config/1.0/">
      <server-params
        min_servers='1'
        min_spare_servers='1'
      />

      <listeners>
        <service
          port="127.0.0.1:6009/tcp" 
          transport="RAW" 
          protocol="SIP/2.10" 
          timeout="5" />
      </listeners>

      <sipohttp>
        <service 
          name="lappisipohttp" 
          host="127.0.0.1" 
          port="6009" />
      </sipohttp>

      <accounts>
        <login id="siptesti"  password="automaatti123" institution="ROOU" delimiter="|" error-detect="enabled" terminator="CR" encoding="utf8" checked_in_ok="1" 
        no_alert="1" sipohttp="lappisipohttp" />
      </accounts>

    <institutions>
      <institution id="ROOU" implementation="ILS" parms="">
        <policy checkin="true" renewal="true" checkout="true" 
          status_update="false" offline="false" 
          timeout="25" 
          retries="5" />
      </institution>
    </institutions>
    </acsconfig>


# Käyttö

Pluginin asennus luo Kohan rajapintaan uuden endpointin /api/v1/contrib/kohasuomi/sipmessages

Tähän endpointiin voi lähettää XML-viestit sisältäen SIP2-viestin SIP2-palvelinta varten. Viestin tulisi noudattaa seuraavan XML-skeeman mukaista rakennetta:


    <?xml version="1.0" encoding="UTF-8"?>

    <!-- sipschema.xsd  $Revision: 1.0 $ support@koha-suomi.fi -->

    <xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema" xmlns:ns1="https://koha-suomi.fi/sipschema.xsd" targetNamespace="https://koha-suomi.fi/sipschema.xsd" elementFormDefault="unqualified" attributeFormDefault="unqualified">

      <xs:element name="sip">

        <xs:complexType>

          <xs:choice>

            <xs:element name="request" type="xs:string"/>

            <xs:element name="response" type="xs:string"/>

            <xs:element name="error" type="xs:string"/>

          </xs:choice>

          <xs:attribute name="login" type="xs:string"/>

          <xs:attribute name="password" type="xs:string"/>

        </xs:complexType>

      </xs:element>

    </xs:schema>



Login- ja password-elementit sisältävät kirjautuvan laitteen tunnukset tunnistautumista varten, request-elementti sisältää SIP2-palvelimelle välitettävän SIP2-protokollan mukaisen viestin. Laitteen tunnusten tulee olla määritettynä SIP2-palvelimen konfiguraatiotiedostoon ja Kohaan Automaatti-asiakastyypille.

Laite, minkä tunnistautuminen vaaditaan puretaan "login:" ja "password" -tiedoista rajapintaan saapuneesta XML:stä. Näillä tiedoilla rakennetaan sip-palvelimelle viesti autentikointia varten.
Sip2-palvelin johon viestit lähetetään, luetaan palvelinymäristössä sijaitsevan SIPconfig-hakemiston sisältämistä konfiguraatiotiedostoista (esim. sipconfig.xml) ja jos tunnus löytyy, autentikointiviesti lähetetään tunnukselle määritettyyn sip2-palvelinosoitteeseen.

Jos autentikointi onnistuu ja sip2-palvelin vastaa "941", lähetetään itse xml:n <request></request> sisällä oleva sip-viesti sip2-palvelimelle. Palautuneesta SIP-viestistä muodostetaan xml-muotoinen paluuviesti.
Paluuviesti välitetään rajapinnan vastaukseksi POST-pyyntöön response bodyssa.

Mikäli sip2-laitteen tunnistautuminen ei onnistu, palautetaan rajapintaan sip-palvelimen palauttama "940"-viesti ja prosessi keskeytyy.


# Virhetilanteet ja loki

SipOHttp-skripti palauttaa rajapinnan kautta seuraavat virheilmoitukset response bodyssa POST-kyselyihin:

-XML validointi epäonnistui skeematiedostoa vastaan: HTTP-virhe 400 viestillä "Invalid Request. Validation failed."
-Jos XML-viestin "login:" tai "password:" parametria vastaavia asetuksia ei löydy XML-konfiguraatiotiedostoista: HTTP 400 "Invalid request. No config found for login device."
-Jos XML-viestin "login:" tai "password:" parametrit ovat puutteelliset: HTTP 400 "Invalid request. Missing login/pw in XML."
-Jos XML-viestin "request" eli SIP2-viesti puuttuu: HTTP 400 "Invalid request. Missing SIP Request in XML."
-Jos sip2-palvelimeen ei saa muodostettua yhteyttä/muu virhe: HTTP 500 "Something went wrong, check the logs."
-Jos SIP2-palvelin aikakatkaisee yhteyden: Paluu-XML, jossa response-osa tyhjä.

Lokit: Sipohttp:n määritykset lokitasosta ja lokin sijainnista määritetään log4perl.conf -tiedostoon.

DEBUG-tasolla sipohttp lokittaa jokaisen rajapintaan saapuvan XML-viestin sisällön ja SIP2-viestiliikenteen SIP2-palvelimen kanssa + virhetilanteet.
ERROR-taso lokittaa vain virheet ja kriittiset virheet (jos SIP2-palvelimeen ei saa luotua socket-yhteyttä).
INFO-taso lokittaa vain sip-palvelimelle/sta kulkevat sanomat.
Lokeihin merkitään myös WARN-tasolla tilanteet, joissa sanoman vastaanotosta vastauksen saamiseen Kohan sip-palvelimelta on kestänyt kauemmin kuin 4 sekuntia.
Lokikonfiguraation/lokituksen puuttuminen ei estä sipohttp:n toimintaa.

Esimerkki log4perl-configuraatiosta:
```
log4perl.logger.sipohttp = DEBUG, SIPoHTTP
log4perl.appender.SIPoHTTP=Log::Log4perl::Appender::File
log4perl.appender.SIPoHTTP.filename=/var/log/koha/sipohttp.log
log4perl.appender.SIPoHTTP.mode=append
log4perl.appender.SIPoHTTP.create_at_logtime=true
log4perl.appender.SIPoHTTP.syswrite=true
log4perl.appender.SIPoHTTP.recreate=true
log4perl.appender.SIPoHTTP.layout=PatternLayout
log4perl.appender.SIPoHTTP.layout.ConversionPattern=[%d{yyyy-MM-dd HH:mm:ss,SSS}] %p %m%n
log4perl.appender.SIPoHTTP.utf8=1
```
