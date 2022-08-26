# openHAB-tagesschau-feed
[Tagesschau](https://www.tagesschau.de/) news feed using the [feed binding](https://www.openhab.org/addons/bindings/feed/) for openHAB.

With this solution you can

* parse strings to urls.
* let Alexa inform you about the last news of the tagesschau feed.
* visit the last news on [https://www.tagesschau.de/](https://www.tagesschau.de/) by clicking inside the sitemap on the link.

## Things

In the things you only have to seed the URL to the news feed:

```
feed:feed:tagesschau [URL="https://www.tagesschau.de/xml/rss2/"]
```

## Items

Additionally to the thing channels you need an string item for parsing the url to the label.

```
String Tagesschau_Feed_LetzteNachricht "Letze Nachricht" {channel="feed:feed:tagesschau:latest-title"}
String Tagesschau_Feed_Nachrichtenbeschreibung "Nachrichtenbeschreibung" {channel="feed:feed:tagesschau:latest-description"}
DateTime Tagesschau_Feed_Nachrichtendatum "Datum der Nachricht" {channel="feed:feed:tagesschau:latest-date"}
String Tagesschau_Feed_LetzteURL "URL" {channel="feed:feed:tagesschau:latest-link"}
Number Tagesschau_Feed_AnzahlEintraege "Anzahl Einträge" {channel="feed:feed:tagesschau:number-of-entries"}
String Tagesschau_Feed_Nachrichtenagentur "Nachrichtenagentur" {channel="feed:feed:tagesschau:description"}
String Tagesschau_Feed_Autor "Autor" {channel="feed:feed:tagesschau:author"}
DateTime Tagesschau_Feed_Veroeffentlichungsdatum "Veröffentlichungsdatum" {channel="feed:feed:tagesschau:last-update"}
String Tagesschau_Feed_TitelNachrichtenagentur "Titel der Nachrichtenagentur" {channel="feed:feed:tagesschau:title"}

String Tagesschau_Feed_News_Link "Link zum Artikel"
```

## Rules

The first rule will give you the possibility to click on the url to visit the last news on [https://www.tagesschau.de/](https://www.tagesschau.de/).

The second rule will give you the possibility that an `Amazon Echo` will inform you about the last news in the feed. Therefore you need to configure the [Amazon Echo Control Binding](https://www.openhab.org/addons/bindings/amazonechocontrol/) and an `Amazo Echo` product. At least you have to create an `<textToSpeech_item>` item which are used for text to speech (tts).

```
rule "open news feed link"
when
    Item Tagesschau_Feed_LetzteURL changed or
    Item Tagesschau_Feed_LetzteNachricht changed or
    Item Tagesschau_Feed_Nachrichtenbeschreibung changed or
    Item Tagesschau_Feed_LetzteURL received update or
    Item Tagesschau_Feed_LetzteNachricht received update or
    Item Tagesschau_Feed_Nachrichtenbeschreibung received update or
    Item Tagesschau_Feed_LetzteURL received command or
    Item Tagesschau_Feed_LetzteNachricht received command or
    Item Tagesschau_Feed_Nachrichtenbeschreibung received command
then
    var url = Tagesschau_Feed_LetzteURL.state.toString()
    logInfo("rules", "Go to {}", url)

    Tagesschau_Feed_News_Link.setLabel(url)
end


rule "new news"
when
    Item Tagesschau_Feed_Nachrichtenbeschreibung changed or
    Item Tagesschau_Feed_Nachrichtenbeschreibung received update or
    Item Tagesschau_Feed_LetzteNachricht received command
then
    var title = Tagesschau_Feed_LetzteNachricht.state
    logInfo("rules", "new news title: {}", title)

    var description = Tagesschau_Feed_Nachrichtenbeschreibung.state
    logInfo("rules", "new news description: {}", description)

    var news = '<speak><amazon:domain name="news"><p>Der Tagesschau-Feed meldet: </p><p>' + title + '</p><break time="1s"/><p>' + description + '</p></amazon:domain></speak>'
    <textToSpeech_item>.sendCommand(news)
end
```

## Sitemap

In addition to the items required by the feed, a webview must be included so that labels from items can be parsed to URLs.

```
Frame label="Nachrichten" {
    Text item=Tagesschau_Feed_LetzteNachricht
    Text item=Tagesschau_Feed_Nachrichtenbeschreibung
    Text item=Tagesschau_Feed_Nachrichtendatum
    Text item=Tagesschau_Feed_LetzteURL
    Text item=Tagesschau_Feed_AnzahlEintraege
    Text item=Tagesschau_Feed_Nachrichtenagentur
    Text item=Tagesschau_Feed_Autor
    Text item=Tagesschau_Feed_Veroeffentlichungsdatum
    Text item=Tagesschau_Feed_TitelNachrichtenagentur
    Text item=Tagesschau_Feed_News_Link

    Webview url="/static/inject_autolinker.html" icon=none
}
```

## HTML

For parsing item labels to URLs you need the [Autolinker.js](https://github.com/gregjacobs/Autolinker.js). Further informations you can get in the [openHAB community](https://community.openhab.org/t/creating-links-to-external-urls-or-other-sitemaps-in-basicui/47067).

I created inside `/etc/openhab/html` a `inject_autolinker.html` file:

```
<!-- 
Place in the sitemap (in every block):
Webview url="/static/inject_autolinker.html"
-->
<head>
    <style>
    </style>
</head>
<body>
    <script src="../static/autolinker/dist/Autolinker.js"></script>
    <script type="text/javascript"> 
        var contents = window.parent.document.body.getElementsByClassName("mdl-form__label");
        var autolinker = new Autolinker( { newWindow: false, truncate: 25 } );        
        
          for (var i=0, max=contents.length; i < max; i++) {
            var content = contents[i];
            content.innerHTML = autolinker.link(content.innerHTML);
          }        
    </script>    
</body>
```

I created inside `/etc/openhab/html/css` a `myStyle.css` file:

```


/* =================================== HEADER FARBE */
.mdl-layout__header {
  background-color: #333;
}

/* =================================== HINTERGRUND FARBE  */

/*  HINTERGRUND STEUERELEMENTE */
.page-content {
  background-color: #1e1e1e;
}
/*  HINTERGRUND Links/Rechts Wrapper */
.mdl-color--grey-100 {
  background-color: #1e1e1e !important;
}

/* =================================== STEUERELEMENTE LINE-HEIGHT */

/* ZEILENHÖHE */
html.ui-layout-condensed .mdl-form__row {
    height: 45px;
}

/* =================================== STEUERELEMENTE FARBE  */

/* STEUERELEMENTE - HINTERGRUND FARBE  */
.mdl-color--white {
    background-color: #383838 !important;
}
/* STEUERELEMENTE - TEXT FARBE  */
.mdl-color-text--grey-700 {
    color: #e0e0e0 !important;
}
/* STEUERELEMENTE - Zwischenstriche Farbe */
.mdl-form__row {
    border-bottom: 1px solid #2d2d2d;
}


/* =================================== SWITCH FARBE OFF */

/* SWITCH Button Farbe */
.mdl-switch__thumb {
  background-color: #fff;
}
/* SWITCH Button Track */
.mdl-switch__track {
    background: #9b4f4f;
}
/* SWITCH Button Button-Farbe */
.mdl-switch__thumb {
  background: #5b5757;
}

/* =================================== SWITCH FARBE ON */
/* SWITCH Button Button-Farbe */
.mdl-switch.is-checked .mdl-switch__thumb {
    background: #d8d8d8;
}
/* SWITCH Button Track-Farbe*/
.mdl-switch.is-checked .mdl-switch__track {
    background: #66d693;
}
/* =================================== FRAME Rundungen */

/* Fenster / Abschnitt Rundungen */
.mdl-form {
  /*+border-radius: 10px;*/
  -moz-border-radius: 3px;
  -webkit-border-radius: 3px;
  -khtml-border-radius: 3px;
  border-radius: 3px;
}

/* =================================== BUTTON FARBE - OFF */

/* BUTTON SELECTED */
BUTTON.mdl-button--accent {
  border: 1px solid #5ddd90;
  /*+box-shadow: 0px 0px !important;*/
  /*-moz-box-shadow: 0px 0px !important;
  -webkit-box-shadow: 0px 0px !important;
  box-shadow: 0px 0px !important;*/
  background-color: #333 !important;
  color: #5ddd90 !important;
}
/* =================================== BUTTON FARBE - ON */

/* BUTTON DESELECTED */
.mdl-form__setpoint BUTTON, .mdl-form__rollerblind BUTTON, .mdl-form__buttons BUTTON {
  color: white;
  background-color: #545454;
  /*+border-radius: 5px;*/
  -moz-border-radius: 2px;
  -webkit-border-radius: 2px;
  -khtml-border-radius: 2px;
  border-radius: 2px;
}

/* =================================== BUTTON FARBE - HOVER */

/* BUTTON HOVER FARBE */
BUTTON.mdl-button:hover {
  background-color: #5ddd90;
}
```

Then I installed

```
npm install autolinker --save
```

to `/etc/openhab/html/node_modules/autolinker`. This folder I moved to `/etc/openhab/html/autolinker`.
