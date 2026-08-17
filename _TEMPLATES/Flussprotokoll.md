<%*
// ── Ausgangspunkt ──────────────────────────────────
const ausgangstyp = await tp.system.suggester(["Situation", "Sozialer Verstärker"], ["situation", "verstaerker"], false, "Ausgangspunkt?");

const situationen = ["Sofortige Zusage", "Nein wird nicht akzeptiert", "Schuldgefuehle", "Provokation", "Staendiges Unterbrechen", "Verantwortung delegieren", "Oeffentliche Kritik", "Grenzueberschreitung", "In Diskussion ziehen", "Rechtfertigung einfordern", "Blossstellung", "Arbeit schlechtmachen", "Schmueckt sich mit meiner Arbeit", "Diskussion nach Entscheidung", "Fangfragen", "Unklare Aufgaben", "Delegation nach oben", "Absprachen nicht einhalten", "Kritik ohne Loesung", "Zeitdruck als Druckmittel", "Gefaelligkeiten einfordern", "Ausweichende Antworten", "Kompetenz infrage stellen", "Ideen nicht ernst nehmen", "Alles zu seinem Problem", "Ja sagen anders handeln", "Gegeneinander ausspielen", "Sofort eskalieren", "Endlos reden", "Nicht zuhoeren", "Kleinigkeit zum Drama"];

const verstaerker = ["Der Gutmenschen-Verteiler", "Der Harmoniebeauftragte", "Der Außenstehende Richter", "Der Verharmloser", "Der Vergleicher", "Der Rollenverteidiger", "Der Zweifler", "Der Konsequenzen-Warner", "Der Schweiger"];

const ausgangsListe = ausgangstyp === "situation" ? situationen : verstaerker;
const ausgangspunkt = await tp.system.suggester(ausgangsListe, ausgangsListe, false, "Welche/r genau?");
const ausgangsLabel = ausgangstyp === "situation" ? "Situation" : "Sozialer Verstärker";

// ── Kette: Verhalten ↔ Reaktion, Runde für Runde ──────────────────────────────────
const verhalten = ["Angeber", "Attention Bitch", "Beleidigte Leberwurst", "Die tote Ziege", "Good girl-boy", "Haar in der Suppe", "I'm walking on Eggshells", "Jakob der Luegner", "Kommste heut nicht", "Master of Puppets", "Pavianarsch", "Probleme wegnehmen", "Ruf Mich An", "Stromberg", "Warmduscher"];

const tools = ["Kaputte Platte", "Fogging", "Nachfragetechnik", "Verzoegerung", "Die Rueckfrage", "Verantwortung zurueckgeben", "DEAR MAN", "GIVE", "FAST", "Grey Rock"];
const bingo = ["Danke dafuer", "Ich nehme das mit", "Interessanter Punkt", "Lass uns das offline besprechen", "Das steht auf dem Radar", "Ich melde mich", "Das passt fuer mich trotzdem nicht", "Meine Entscheidung steht", "Ich habe dazu nichts hinzuzufuegen", "Das sehe ich anders", "Das ist nicht meine Verantwortung", "Kann sein", "Ich war noch nicht fertig", "Ich komme gleich zu deinem Punkt", "Das funktioniert fuer mich nicht", "Das muss ich nicht begruenden", "Interessant dass du das gerade ansprichst", "Danke fuer dein Feedback", "Da war noch jemand beteiligt", "Die Entscheidung steht", "Was genau moechtest du damit herausfinden", "Ich brauche noch eine konkrete Angabe zu", "Das entscheidest du", "Das war so nicht abgesprochen", "Was schlaegst du konkret vor", "Ich brauche trotzdem Zeit", "Tut mir leid da kann ich dir nicht helfen", "Das war noch keine Antwort auf meine Frage", "Ich moechte dass das noch mal aufgegriffen wird", "Das betrifft gerade jemand anderen", "Was genau bedeutet das jetzt konkret", "Das klaere ich direkt mit der Person", "Lass uns das in Ruhe klaeren", "Ich fasse mal zusammen damit wir weiterkommen", "Ich moechte das noch mal wiederholen bis es ankommt", "Das kriegen wir hin"];
const reaktionen = tools.concat(bingo);

const kette = [];
let weiter = true;
while (weiter) {
  const v = await tp.system.suggester(verhalten, verhalten, false, "Verhalten (Gegenüber) – Runde " + (kette.length + 1));
  const r = await tp.system.suggester(reaktionen, reaktionen, false, "Reaktion (ich) – Runde " + (kette.length + 1));
  kette.push([v, r]);
  const weiterWahl = await tp.system.suggester(["✓ Noch eine Runde", "FERTIG — Fazit schreiben"], [true, false], false, "Weiter? (bisher " + kette.length + " Runde(n))");
  weiter = weiterWahl;
}

// ── Archetyp (optional) ──────────────────────────────────
const archetypen = ["(unbekannt)", "Papa Schlumpf", "Der Schnorrer", "Der Micromanager", "Der Drama-Magnet", "Der Maertyrer", "Der Blender", "Der Besserwisser-Chef", "Der Phantomarbeiter", "Der Harmonie-Diktator", "Der Strippenzieher", "Der Platzhirsch", "Der Noergler", "Der Ausweicher", "Der Opferkoenig", "Der Suendenbock-Sucher", "Der Dampfwalzen-Typ", "Der Radfahrer", "Der Trittbrettfahrer", "Der Nachtreter", "Der Phrasendrescher", "Der Schoenredner", "Der Krisenjunkie", "Das Chamaeleon", "Der Teflon-Mensch", "Schlaubi Schlumpf", "Der Mansplainer", "Der Windbeutel", "Die Schlumpfine", "Der Babo", "Das Faehnchen-im-Wind", "Der Credit-Sammler", "Der Paragraphenreiter"];
const archetyp = await tp.system.suggester(archetypen, archetypen, false, "Archetyp, falls bekannt");

// ── Properties direkt über Obsidians API setzen, unabhängig davon, was schon in der Datei steht ──────────────────────────────────
// Kurze Wartezeit, damit das Timestamps-Plugin (front-matter-timestamps) zuerst fertig schreiben kann
await new Promise((resolve) => setTimeout(resolve, 1500));
const zielDatei = tp.file.find_tfile(tp.file.title);
await app.fileManager.processFrontMatter(zielDatei, (fm) => {
  fm["typ"] = "fallstudie";
  fm["created"] = tp.date.now("YYYY-MM-DD");
  fm["ausgangstyp"] = ausgangsLabel;
  fm["ausgangspunkt"] = "[[" + ausgangspunkt + "]]";
  fm["archetyp"] = archetyp === "(unbekannt)" ? "unbekannt" : "[[" + archetyp + "]]";
  fm["runden"] = kette.length;
  fm["status"] = "";
});
-%>
# <% tp.file.title %>

## Ausgangspunkt
<% ausgangsLabel %>: ![[<% ausgangspunkt %>#Funktion]]

## Kette
<%* for (let i = 0; i < kette.length; i++) { -%>
**Runde <% i + 1 %>**
Verhalten (Gegenüber): ![[<% kette[i][0] %>#Kurzbeschreibung]]
Reaktion (ich): ![[<% kette[i][1] %>#<% tools.includes(kette[i][1]) ? "Prinzip" : "Funktion" %>]]

<%* } -%>
## Fazit
- Was war zielführend?
