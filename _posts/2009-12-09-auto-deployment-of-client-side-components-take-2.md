---
layout: post
title: "Auto Deployment of Client Side Components – take 2"
date: 2009-12-09 11:11:03
categories: ["Archive"]
tags: ["Add-Ins", "Client Tier", "COM", "Deployment", "Extensibility", "NAV 2009 SP1", "Service Tier", "Web Services"]
permalink: /2009/12/09/auto-deployment-of-client-side-components-take-2/
---

Updated the link to the ComponentHelper msi on 12/11/2009

Please read my first post about auto deployment of Client side components [here](http://blogs.msdn.com/freddyk/archive/2009/09/19/auto-deployment-of-client-side-components.aspx) before reading this.

As you know, my first auto deployment project contained a couple of methods for automatically adding actions to pages, but as one of my colleagues in Germany (Carsten Scholling) told me, it would also need to be able to add fields to tables programmatically in order to be really useful.

In fact, he didn’t just tell me that it should do so, he actually send me a couple of methods to perform that.

The method signatures are:

`AddToTable(TableNo : Integer;FieldNo : Integer;VersionList : Text[30];FieldName : Text[30];FieldType : Integer;FieldLength : Integer;Properties : Text[800]) : Boolean`

and

`AddTheField(TableNo : Integer;FieldNo : Integer;FieldName : Text[30];FieldType : Integer;FieldLength : Integer) SearchLine : Text[150]`

And it can be used like:

```
// Add the fields
SearchLineLat := ComponentHelper.AddTheField(DATABASE::Customer, 66030, 'Latitude',  FieldRec.Type::Decimal, 0);
SearchLineLong := ComponentHelper.AddTheField(DATABASE::Customer, 66031, 'Longitude',  FieldRec.Type::Decimal, 0);
```

which just add’s the fields without captions or

```
// Add Latitude to Customer table
```

```
ComponentHelper.AddToTable(DATABASE::Customer, 66030, 'VirtualEarthDemo1.01', 'Latitude', FieldRec.Type::Decimal, 0,
'CaptionML=[DAN=Breddegrad;DEU=Breitengrad;ENU=Latitude;ESP=Latitud;FRA=Latitude;ITA=Latitudine;NLD=Breedte];DecimalPlaces=6:8');
```

Remember, that the table will be left uncompiled after doing this.

AddToTable actually calls AddTheField and after that it modifies the metadata to set the caption on the field:

**`AddToTable(TableNo : Integer;FieldNo : Integer;VersionList : Text[30];FieldName : Text[30];FieldType : Integer;FieldLength : Integer;Properties : Text[800]) : Boolean`  
**

```
changed := FALSE;
SearchLine := AddTheField(TableNo, FieldNo, FieldName, FieldType, FieldLength);
IF SearchLine <> " THEN
BEGIN
GetTableMetadata(TableNo, Metadata);
IF AddToMetadataEx(TableNo, Object.Type::Table, Metadata, SearchLine, ", ';' + Properties, TRUE, FALSE) THEN BEGIN
SetTableMetadata(TableNo, Metadata, VersionList);
changed := TRUE;
END;
END;
```

AddTheField is the actual “magic”:

```
AddTheField(TableNo : Integer;FieldNo : Integer;FieldName : Text[30];FieldType : Integer;FieldLength : Integer) SearchLine : Text[150]
Field.SETRANGE(TableNo, TableNo);
Field.SETRANGE("No.", FieldNo);
SearchLine := ";
```

```
IF Field.ISEMPTY THEN BEGIN
Field.TableNo := TableNo;
Field."No." := FieldNo;
Field.FieldName := FieldName;
Field.Type := FieldType;
Field.Class := Field.Class::Normal;
Field.Len := FieldLength;
Field.Enabled := TRUE;
Field.INSERT;
```

  `Field.FINDFIRST;`

  

```
Len[1] := 4;
Len[2] := 20;
Len[3] := 15;
```

  

```
IF STRLEN(FORMAT(FieldNo))   > Len[1] THEN Len[1] := STRLEN(FORMAT(FieldNo));
IF STRLEN(FieldName)         > Len[2] THEN Len[2] := STRLEN(FieldName);
IF STRLEN(Field."Type Name") > Len[3] THEN Len[3] := STRLEN(Field."Type Name");
```

  

```
SearchLine := '    { ' + PADSTR(FORMAT(FieldNo), Len[1]) + ';  ;' +
PADSTR(FieldName, Len[2]) + ';' + PADSTR(Field."Type Name", Len[3]);
END;
```

The new ComponentHelper 1.03 msi can be downloaded [here](http://www.freddy.dk/ComponentHelper1.03.zip) and my upcoming posts (e.g. Edit In Excel R2) will require this. If you only want to download the objects you can do so [here](http://www.freddy.dk/ComponentHelperObjects1.03.zip) (there is no changes in the NAVAddInHelper source (compared to the first post – that can be downloaded [here](http://www.freddy.dk/NAVAddInHelper1.01.zip)).

Enjoy

_**Freddy Kristiansen**  
__PM Architect  
Microsoft Dynamics NAV_
