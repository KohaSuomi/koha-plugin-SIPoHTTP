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


#Käyttö

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

ks. https://tiketti.koha-suomi.fi/projects/koha-suomen-dokumentaatio/wiki/SipOhttp
ja https://tiketti.koha-suomi.fi/projects/koha-suomen-dokumentaatio/wiki/SIP2-tunnusten_k%C3%A4sittelyohje