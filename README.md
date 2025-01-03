# BT-KAT-OWI
Import tool for the German ["Bundeseinheitlichen Tatbestandskatalog" (BT-KAT-OWI)](https://www.kba.de/DE/Themen/ZentraleRegister/FAER/BT_KAT_OWI/btkat_node.html) into Microsoft Excel or [Microsoft PowerBI](https://powerbi.microsoft.com). This solution is based on the information from the [bkat-owi repository](https://github.com/jomo/bkat-owi/blob/master/README.md).

# Data
The Excel file contains the current, valid data, which are represented by the following columns:
1. TBNR
2. Tatbestand
3. Paragraphen
4. FaP
5. PKT
6. EuroGesamt
7. Klammer-a
8. Klammer-b
9. Klammer-c
10. GUELTIG-bis
11. GUELTIG-von
12. FV
13. Konkretisierungsindikator
14. KonkretisierungsindikatorText
15. Klassifizierung
16. KlassifizierungsText
17. Tabelle
18. Untergrenze
19. Obergrenze

[Description of the record structure of the database version of the
Uniform federal catalog of facts (status: 01.03.2007)](https://fragdenstaat.de/anfrage/alle-versionen-und-unterlagen-des-bundeseinheitlichen-tatbestandskatalogs/595927/anhang/SatzbeschreibungBETBK-gltigab010307__konvertiert.pdf)

# Query the data
When you click the _"Update All"_ button, current data is downloaded from the website: https://www.kba.de/DE/Themen/ZentraleRegister/FAER/BT_KAT_OWI/bet_datenbank_22082024_txt.asc?__blob=publicationFile&v=7 and converted.

Currently the file name is _"bet_datenbank_22082024_txt"_ which will surely change in the future. When that happens, the query will no longer work. In this case, this must be taken into account in the link.

The data is queried using [Microsoft Power Query](https://docs.microsoft.com/power-query) for Excel.
```
let
    Quelle = Csv.Document(Web.Contents("https://www.kba.de/DE/Themen/ZentraleRegister/FAER/BT_KAT_OWI/bet_datenbank_22082024_txt.asc?__blob=publicationFile&v=7"),[Delimiter="^", Columns=30, Encoding=28591]),
    KlassifizierungsTextRecord = [
            0 = "sonstige",
            1 = "Lichtzeichen",
            2 = "Behinderung",
            3 = "Gefährdung",
            4 = "Unfall",
            5 = "Halt- und Parkverstoß",
            6 = "Geschwindigkeitsüberschreitung",
            7 = "Überladung",
            8 = "Alkohol und Rauschmittel",
            9 = "Abstandsmessung",
            M = "Messangabe"
        ],
    KonkretisierungsindikatorTextRecord = [
            0 = "",
            1 = "<>",
            2 = "+)",
            3 = "Kombination aus +) und <>",
            4 = "*); **); ***) u.s.w.",
            5 = "Kombination aus *) und +) ",
            6 = "Kombination aus *) und <> ",
            7 = "Kombination aus *) und <> und +)"
        ],
    TypenAenderung = Table.TransformColumnTypes(Quelle,{{"Column1", type text}, {"Column2", type text}, {"Column3", type text}, {"Column4", type text}, {"Column5", type text}, {"Column6", type text}, {"Column7", type text}, {"Column8", type text}, {"Column9", type text}, {"Column10", Int64.Type}, {"Column11", Int64.Type}, {"Column12", Int64.Type}, {"Column13", type text}, {"Column14", type text}, {"Column15", type text}, {"Column16", type date}, {"Column17", type date}, {"Column18", type text}, {"Column19", type text}, {"Column20", type text}, {"Column21", type text}, {"Column22", type text}, {"Column23", type text}, {"Column24", type text}, {"Column25", type text}, {"Column26", type text}, {"Column27", type text}, {"Column28", type text}, {"Column29", type text}, {"Column30", type text}}),
    Spaltenumbenennung = Table.RenameColumns(TypenAenderung,{{"Column1", "TBNR"}, {"Column2", "Tatbestand-z1"}, {"Column3", "Tatbestand-z2"}, {"Column4", "Tatbestand-z3"}, {"Column5", "Tatbestand-z4"}, {"Column6", "Tatbestand-z5"}, {"Column7", "Paragraphen-z1"}, {"Column8", "Paragraphen-z2"}, {"Column9", "FaP"}, {"Column10", "PKT"}, {"Column11", "Euro"}, {"Column12", "Euro-zus"}, {"Column13", "Klammer-a"}, {"Column14", "Klammer-b"}, {"Column15", "Klammer-c"}, {"Column16", "GUELTIG-bis"}, {"Column17", "GUELTIG-von"}, {"Column18", "FV"}, {"Column19", "Konkretisierungsindikator"}, {"Column20", "Klassifizierung"}, {"Column29", "Paragraphen-z1-Druck"}, {"Column30", "Paragraphen-z2-Druck"}, {"Column24", "Tatbestand-z1-Druck"}, {"Column25", "Tatbestand-z2-Druck"}, {"Column26", "Tatbestand-z3-Druck"}, {"Column27", "Tatbestand-z4-Druck"}, {"Column28", "Tatbestand-z5-Druck"}, {"Column21", "Tabelle"}, {"Column22", "Untergrenze"}, {"Column23", "Obergrenze"}}),
    TatbestandZusammen = Table.AddColumn(Spaltenumbenennung, "Tatbestand", each Text.Combine({[#"Tatbestand-z1"], [#"Tatbestand-z2"], [#"Tatbestand-z3"], [#"Tatbestand-z4"], [#"Tatbestand-z5"]}," "), type text),
    ParagraphenZusammen = Table.AddColumn(TatbestandZusammen, "Paragraphen", each Text.Combine({[#"Paragraphen-z1"],[#"Paragraphen-z2"]}, " "), type text),
    EuroGesamt = Table.AddColumn(ParagraphenZusammen, "EuroGesamt", each [Euro] + ([#"Euro-zus"] * 0.01), type number),
    KlassifizierungsText = Table.AddColumn(EuroGesamt, "KlassifizierungsText", each Record.Field(KlassifizierungsTextRecord, [Klassifizierung]), type text),
    KonkretisierungsindikatorText = Table.AddColumn(KlassifizierungsText, "KonkretisierungsindikatorText", each if Text.Trim([Konkretisierungsindikator]) = "" then "" else Record.Field(KonkretisierungsindikatorTextRecord, [Konkretisierungsindikator]), type text),
    SpaltenAnordung = Table.ReorderColumns(KonkretisierungsindikatorText,{"TBNR", "Tatbestand", "Tatbestand-z1", "Tatbestand-z2", "Tatbestand-z3", "Tatbestand-z4", "Tatbestand-z5", "Paragraphen", "Paragraphen-z1", "Paragraphen-z2", "FaP", "PKT", "EuroGesamt", "Euro", "Euro-zus", "Klammer-a", "Klammer-b", "Klammer-c", "GUELTIG-bis", "GUELTIG-von", "FV", "Konkretisierungsindikator", "KonkretisierungsindikatorText", "Klassifizierung", "KlassifizierungsText", "Tabelle", "Untergrenze", "Obergrenze", "Tatbestand-z1-Druck", "Tatbestand-z2-Druck", "Tatbestand-z3-Druck", "Tatbestand-z4-Druck", "Tatbestand-z5-Druck", "Paragraphen-z1-Druck", "Paragraphen-z2-Druck"}),
    SpaltenEntfernung = Table.RemoveColumns(SpaltenAnordung,{"Tatbestand-z1", "Tatbestand-z2", "Tatbestand-z3", "Tatbestand-z4", "Tatbestand-z5", "Paragraphen-z1", "Paragraphen-z2", "Tatbestand-z1-Druck", "Tatbestand-z2-Druck", "Tatbestand-z3-Druck", "Tatbestand-z4-Druck", "Tatbestand-z5-Druck", "Paragraphen-z1-Druck", "Paragraphen-z2-Druck", "Euro", "Euro-zus"}),
    ZeilenGruppierung = Table.Group(SpaltenEntfernung, {"TBNR"}, {{"LetzterGueltigerEintrag", each Table.First(Table.Sort(_, {"GUELTIG-bis", Order.Descending}))}}),
    LetzterGueltigerEintragErweitern = Table.ExpandRecordColumn(ZeilenGruppierung, "LetzterGueltigerEintrag", {"Tatbestand", "Paragraphen", "FaP", "PKT", "EuroGesamt", "Klammer-a", "Klammer-b", "Klammer-c", "GUELTIG-bis", "GUELTIG-von", "FV", "Konkretisierungsindikator", "KonkretisierungsindikatorText", "Klassifizierung", "KlassifizierungsText", "Tabelle", "Untergrenze", "Obergrenze"}, {"Tatbestand", "Paragraphen", "FaP", "PKT", "EuroGesamt", "Klammer-a", "Klammer-b", "Klammer-c", "GUELTIG-bis", "GUELTIG-von", "FV", "Konkretisierungsindikator", "KonkretisierungsindikatorText", "Klassifizierung", "KlassifizierungsText", "Tabelle", "Untergrenze", "Obergrenze"}),
    TypAenderung2 = Table.TransformColumnTypes(LetzterGueltigerEintragErweitern,{{"Tatbestand", type text}, {"Paragraphen", type text}, {"FaP", type text}, {"PKT", Int64.Type}, {"EuroGesamt", type number}, {"Klammer-a", type text}, {"Klammer-b", type text}, {"Klammer-c", type text}, {"GUELTIG-bis", type date}, {"GUELTIG-von", type date}, {"FV", type text}, {"Konkretisierungsindikator", type text}, {"Klassifizierung", type text}, {"Tabelle", type text}, {"Untergrenze", type text}, {"Obergrenze", type text}, {"KlassifizierungsText", type text}, {"KonkretisierungsindikatorText", type text}})
in
    TypAenderung2
```
# Using the data
The original dataset contains historical information for each *Tatbestand*. This information is preserved until the step *SpaltenEntfernung*. The following steps group the dataset and return the most recent entry per *Tatbestand*.

# License
Traffic code offense data (Tatbestandskatalog) is in the public domain according to [§ 5 UrhG](https://www.gesetze-im-internet.de/urhg/__5.html). The repository is released under the [MIT](https://choosealicense.com/licenses/mit/).
