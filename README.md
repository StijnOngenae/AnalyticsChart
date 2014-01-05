Stappenplan Project 
=========================
Om ervoor te zorgen dat u de views kan bekijken van uw site moeten enkele
opties in orde gebracht worden.

Google Analytics
---------------------

Eerst en vooral zal u een account moeten aanmaken bij Google Analytics "https://www.Google.be/intl/nl/analytics/".
Hier zal u een trackingcode ontvangen voor uw site in deze vorm: 'UA-XXXXXXXX-X'. 
Deze code moet u kopiëren in de algemene site-instellingen die u kan terugvinden bij "Site beheren".
Hierdoor zal Google Analytics een aantal gegevens bijhouden van uw site.

Google Cloud Console
-------------------------

Het volgende die zal gedaan moeten worden, is een project aanmaken in de Google Cloud Console.
Met dit project kunt u requests sturen naar de gegevens die Google Analytics van uw site bijhoudt. 
Een project kan privaat of openbaar ingesteld worden. 
Dit zal belangrijk worden om te bepalen of requests van geautoriseerde gebruikers moet komen.
In uw project moet Google Analytics Api worden ingeschakeld.
Dit doet u door op de Google Cloud Console website, op het onderdeel Api's te klikken en Analytics API aan te zetten.
Daarnaast zal u bij het onderdeel Credentials twee zaken moeten regelen.
Eerst en vooral maakt u een nieuw client ID aan voor een webapplicatie.
Hier moet u bij "Authorized Javascript origins" de domeinnaam van uw site zetten.
U heeft hiernaast ook een access key nodig, een zogenaamde API key. In dit geval gebruiken we een API Key voor een browser applicatie.
Bij het aanmaken van API Key, kan u bepalen welke websites naar de gegevens van uw site mogen bekijken. Vult u niemand in dan geeft u toestemming aan elke site om uw gegevens te bekijken.
Als al deze stappen gevolgd werden, kan u beginnen aan het schrijven van de code die zal leiden tot een grafiek die het aantal bezoeken van uw site gedurende een bepaalde periode voorstelt.

Voor dit voorbeeld gebruiken we 3 bestanden.

_Main.html_ waar de grafiek op zal worden afgebeeld.

_Authentication.js_ waar we verifiëren of de gebruiker toegang heeft tot uw gegevens. Dit is niet nodig bij een openbaar toegangkelijk project.

_Visualisation.js_ waar we de gegevens ophalen en omzetten in een Google Chart.


 Main.html
--------------
In de htmlfile worden de buttons en het script ingeladen.
Maar voor u dit doet, zullen we in de head een verwijzing zetten naar "https://www.google.com/jsapi" zodat we de verschillende API's van Google kunnen laden via JavaScript.
In de body kan u een hoofding maken met daaronder een div waarin 2 buttons (die zorgen voor de autho-
risatie en de visualisatie) zitten.
Daarnaast worden 2 javascripts geladen (die voor de Authenticatie en Visualisatie).
Als laatste moet ook een link gelegd worden met de client library en geven we ook een onload handler mee die ervoor zorgt dat de functie handleClientLoad getriggerd wordt. Indien u met een openbaar project werkt, hoeft u enkel de functie handleClientLoad te vervangen door makeApiCall. Daarnaast heeft u ook de Authentication.js file en de 2 buttons niet nodig.

    <!DOCTYPE>
    <html>      
    <head>      
    <title>AnalyticsChart</title>      
    <script type="text/javascript" src=https://www.google.com/jsapi></script>      
    </head>      
    <body>      
    <h1>Van Google Analytics naar een Google Chart</h1>      
    <div id="chart_div"></div>     
  
    <button id="authorize-button" style="visibility: hidden">Autoriseren</button>   
    <br/>      
    <button id="make-api-call-button" style="visibility: hidden">Toon Google Chart</button>      
  
    <script src="Authentication.js"></script>      
    <script src="Visualisation.js"></script>      
  
    <script src="https://apis.google.com/js/client.js?onload=handleClientLoad"</script      
    </body      
    </html      

Authentication.js
----------------------
Eerst en vooral worden in dit bestand de clientId, apiKey en de scopes aangemaakt.
Zowel de clientId en apiKey kan men terugvinden bij het project dat u aangemaakt hebt in de Google Cloud Console.
De scope is readonly zodat men uw site niet kan aanpassen (https://www.googleapis.com/auth/analytics.readonly).
Na het zetten van de API Key, kijken we of de gebruiker al dan niet toestemming geeft om zijn Analytics gegevens te mogen gebruiken. We gebruiken hiervoor de checkAuth functie de google api (gapi).

handleAuthResult bekijkt of de gebruiker toestemming heeft gegeven. Is dit het geval dan laden we de Google Analytics Api in door middel van "gapi.client.load" waar we de api en de versie van deze api ingeven. De functie die we meegeven, zorgt ervoor dat de button "Autoriseren" verdwijnt en de andere knop tevoorschijn komt.
Aan de knop wordt dan een onclick event gebonden die de functie "makeApiCall()" oproept.

    var clientId = '663658222418-41alrf9qinerlds3b71d93ga777jdfki.apps.googleusercontent.com';  
    var apiKey = 'AIzaSyANDBwPvO1iEdHrsrgj_W3fe_NK6lu4gTY';  
    var scopes = 'https://www.googleapis.com/auth/analytics.readonly';  
  
    // Wordt opgeropen als de client library werd geladen  
    function handleClientLoad() {  
    // 1. De API key wordt 'geset'  
    gapi.client.setApiKey(apiKey);  

    // 2. Kijkt of de user geautoriseert is  
    window.setTimeout(checkAuth, 1);  
    }  

    function checkAuth() {  
    // Hierbij wordt de Google Accounts Service opgeropen om de gebruiker te laten inloggen  
    // die gegevens worden dan doorgespeeld naar de handleAuthResult functie  
    gapi.auth.authorize({ client_id: clientId, scope: scopes, immediate: true }, handleAuthResult);  
    }  

    function handleAuthResult(authResult) {  
    if (authResult) {  
    // De gebruiker is geautoriseert  
    // Laad de Analytics Client.  
    loadAnalyticsClient();  
    } else {  
    // De gebruiker is niet geautoriseert  
    handleUnAuthorized();  
    }  
    }  

    function loadAnalyticsClient() {  
    // Laad de Analytics Client en roep de functie handleAuthorized op  
    gapi.client.load('analytics', 'v3', handleAuthorized);  
    }  

    function handleAuthorized() {  
    var authorizeButton = document.getElementById('authorize-button');  
    var makeApiCallButton = document.getElementById('make-api-call-button');  

    // Toon de Toon Google Chart button en verberg de Autoriseren button  
    makeApiCallButton.style.visibility = '';  
    authorizeButton.style.visibility = 'hidden';  

    // makeApiCall functie wordt opgeroepen als de Toon Google Chart button wordt aangeklikt  
    makeApiCallButton.onclick = makeApiCall;  
    }  

    function handleUnAuthorized() {  
    var authorizeButton = document.getElementById('authorize-button');  
    var makeApiCallButton = document.getElementById('make-api-call-button');  

    // Toon de Autoriseren button en verberg de Toon Google Chart button  
    makeApiCallButton.style.visibility = 'hidden';  
    authorizeButton.style.visibility = '';  

    // handleAuthClick functie wordt opgeropen als de Autoriseren button wordt aangeklikt  
    authorizeButton.onclick = handleAuthClick;  
    }  

    function handleAuthClick(event) {  
    gapi.auth.authorize({ client_id: clientId, scope: scopes, immediate: false }, handleAuthResult);  
    return false;  
    }  


Visualisation.js
--------------------

In dit bestand gaan we eerst en vooral de client inladen om Google Charts te kunnen tekenen. De functie 'makeApiCall()' wordt opgeroepen en doorloopt het proces van een Google Analytics Request. In dit voorbeeld willen we graag de gegevens van 10 dagen verzamelen. Daarvoor hebben we 2 variabelen nodig: today en aTimeAgo. In de code kan u zien hoe we deze 2 variabelen bepalen.
De gegevens voor onze grafiek kunnen we verkrijgen door een API request te versturen. Hiervoor hebben een aantal gegevens nodig. De start- en einddatum zijn al bepaald. Daarnaast hebben we ook een viewid nodig. Die kan je terugvinden in de tab 'Beheerder' op uw Google Analytics Account. Bij de kolom 'Weergave' kan je bij 'Instellingen weergeven' een ID van dataweergave vinden. Dit moet je onder de vorm 'ga:XXXXXXXXX', waarbij de x'en uw getal zijn, bij ids zetten. Bij metrics en dimensions kan je bepalen wat je terug wil krijgen van gegevens. Bent u niet zeker? Gebruik dan de query explorer van Google zelf: "http://ga-dev-tools.appspot.com/explorer/"

Het object dat je terugkrijgt is een JSON-object waarop we de 'drawchart' functie op los laten. Deze functie pakt de resultaten van het object en giet ze in een Google Chart. Eerst en vooral bepalen we de kolommen met de 'addColumn' methode. Hierna worden de rijen toegevoegd. De substrings dat we gebruiken is om de datum in het juiste formaat te weergeven. De opties worden bepaald. Zie "https://developers.google.com/chart/interactive/docs/gallery" voor een lijst van de verschillende opties per soort grafiek. In ons voorbeeld kiezen we voor een Linechart. De grafiek wordt gemaakt door een Linechart object in te laden op de div die we hiervoor aangesteld hebben.

    // De visualisatie APi moet geladen worden om een grafiek te kunnen tekenen.  
    google.load('visualization', '1.0', { 'packages': ['corechart'] });  
    // Wanneer de 'Make API Call' button wordt aangeklikt wordt deze functie doorlopen  
    function makeApiCall() {  
    queryCoreReportingApi();  
    }  

    function queryCoreReportingApi() {  
    console.log('Querying Core Reporting API.');  
    var today = new Date('2013-12-2');  
    var dd = today.getDate();  
    var mm = today.getMonth() + 1; //Januari is 0, daarom doen we getMonth()+1  
    var aTimeAgo = today;  
    var dd = today.getDate();  
    aTimeAgo.setDate(dd - 10);  
    var Dd = today.getDate();  
    var Mm = aTimeAgo.getMonth() + 1;   
    var Yyyy = aTimeAgo.getFullYear();  
    var yyyy = today.getFullYear();  
    //Omzetten van de datum naar een gepast formaat.  
    if (dd < 10) { dd = '0' + dd } if (mm < 10) { mm = '0' + mm } today = yyyy + '-' + mm + '-' + dd;   
    if (dd < 10) { dd = '0' + dd } if (mm < 10) { mm = '0' + mm } aTimeAgo = Yyyy + '-' + Mm + '-' + Dd;  
    // De Analytics Service object wordt gebruikt om de Core Reporting APi naar een query om te zetten  
    gapi.client.analytics.data.ga.get({  
    'ids': 'YOUR VIEW ID',  
    'start-date': aTimeAgo,  
    'end-date': today,  
    'dimensions' : 'ga:date',  
    'sort' : 'ga:date',  
    'metrics': 'ga:visits',  
    'max-results' : 100  
    }).execute(handleCoreReportingResults);  
    }  

    function handleCoreReportingResults(results) {  
    if (results.error) {  
    console.log('There was an error querying core reporting API: ' + results.message);  
    } else {  
    google.setOnLoadCallback(drawChart(results));  
    }  
    }  

    function drawChart(result) {  
    // Maak de datatable aan  
    var data = new google.visualization.DataTable();  
    data.addColumn('string', parseInt(result.columnHeaders[0].name));  
    data.addColumn('number', 'Visits');  
    for (var index = 0; index < result.rows.length; index++) {  
    data.addRow([(result.rows[index][0]).substring(6, 8) + '-' + (result.rows[index][0]).substring(4, 6) + '-' + (result.rows[index][0]).substring(0, 4), parseInt(result.rows[index][1])]);  
    }  

    // Hier worden de grafiekopties beschreven  
    var options = {  
    'title': 'Page Traffic',  
    'vAxis': {title: "#Visitors"},  
    'width': 400,  
    'height': 300,  
    'hAxis': {title: "Date",}  
    };  

    // Instantieer de grafiek en toon hem op het scherm  
    var chart = new google.visualization.LineChart(document.getElementById('chart_div'));  
    chart.draw(data, options);  
    }  

**Dit stappenplan kan u volgen om tot een grafiek te komen die het aantal bezoekers van uw site voorstelt voor een bepaalde periode.**
