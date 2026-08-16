# Equipment — User Guide

Management of climbing equipment and PPE inspections: which part, which item,
inspected when, retired when.

## Basic concepts

**Part** is the model: a carabiner from a particular manufacturer, with name,
material, colour and standard. One part describes many identical pieces.

**Item** is a single piece of it — with its own number, its own serial number,
its own purchase date and its own storage location. Every item refers to a
part.

**Inspection** belongs to the item. If the item is deleted, its inspections go
with it.

Numbers for items and parts are assigned by the program on creation. They
cannot be changed.

## The tabs

| Tab | Contents |
| --- | --- |
| Items | one item with its fields, the associated part, the current inspection and the pictures |
| Parts | one part with its fields and the picture |
| Search | search across items, parts and inspections |
| Options | users, language, clients, appearance, date format, visibility settings |
| Info | platform, storage locations, data holdings, licences |
| Debug | log (only if switched on in the Options) |

## Getting started

1. In the **Parts** tab click *New* and enter the model: name, manufacturer,
   order no., material, standards. The number is assigned by the program.
2. In the **Items** tab click *New* and enter the number of the part you have
   just created in *Part number*. Instead of the number, the part name also
   works — the program looks it up and fills in the number.
3. Add purchase date, service life, storage location and serial number.

Saving happens as soon as a field is left. There is no Save button.

## Adjusting the view

The small triangles along the left edge collapse sections: the part details,
the further item fields, the inspections and the pictures. Collapsed, the
first line of each stays visible so you can see what it is about. On a phone
this pays off — whatever is not needed disappears and the rest fits on the
screen.

The state is kept, even after quitting.

## Language and appearance

Two dropdowns sit in the Options:

**Language** lists the available translations. If only "default" is offered
there are none — everything then appears as written in the program.

**Colour scheme**: light, dark, or follow the system. Both apply per client.

## Paging

The bar along the top edge: to the beginning, five back, one back, one
forward, five forward, to the end. The small field next to *New* jumps
straight to a number.

The magnifier symbol in the middle toggles between the search result and the
full stock. When it is highlighted in orange, you are paging through the
search result.

## Data

**Retirement date** is calculated from the purchase date and the service life.
If you enter it by hand, the value you entered stays.

**Date entry** understands several notations: `2024-12-31`, `31.12.2024`,
`Dec 2024`, `2024`. Missing parts are filled in with the start of the month.
Display uses the format set in the Options.

**Pictures** can be loaded from a file or taken from the clipboard: copy the
image address on the manufacturer's page, then click *Paste* in the app. Item
and part each have their own picture.

**QR code** is generated from the *QR code content* field and is redrawn every
time it is displayed.

## Search

Choose a search field, enter a term, click *Search*. If there are hits, the
view switches to the Items tab.

- `Helmet` finds the text at any position
- `2024` finds the whole of 2024
- `>=2024` finds everything from 1 January 2024 onwards
- `<2024` finds everything before that
- *All fields* searches items, parts and inspections at the same time
- *Due for retirement* shows all items whose retirement date has passed

*Clear search* shows the entire stock again.

## Inspections

The Items tab shows only the current inspection. *New* creates one with
today's date; if a user name is entered in the Options, it appears in the
remark. If there are older inspections, *History* leads to them.

## NFC

The *NFC read* and *NFC write* buttons are next to the serial number. If they
are greyed out, no NFC is available — the reason is given in the Info tab.

On a phone it is enough to hold the device against the tag; no button press is
needed. On the desktop a connected USB card reader is required.

What a scan does depends on the cursor:

- Cursor in the **empty** serial number field: the number read is entered
- otherwise: the item with that serial number is searched for and displayed

*NFC write* writes the displayed serial number to the tag. If the tag contains
foreign data, you are asked for confirmation. Brand-new tags must be formatted
first — the corresponding switch in the Options allows this. Formatting
overwrites everything else that is on the chip.

## Clients

Every client has its own data: its own database, its own settings. In the
Options a client can be created, switched or deleted. On startup the one used
last is opened.

The last remaining client is not deleted, only emptied — without a client the
program would have nothing to display.

## Backing up and transferring

In the menu on the left-hand edge:

- **Create client backup** packs settings and database into a ZIP file.
- **Import client backup** reads them back in, replacing this client's stock in
  the process.
- **Migrate clients** transfers items and parts from one client to another.
  Records that already exist are skipped; those taken over are given new,
  consecutive numbers in the target. Inspections travel with their item.

On the desktop the storage location is chosen in the save dialog, on the phone
in the system's share dialog (where "Save as" is one of the entries).

## PDF

**Export PDF** in the menu asks for:

- **Scope**: the current item, the search result, or everything
- **Sorting**: two levels, the second applies in the event of a tie
- **Compact**: tighter spacing and smaller type, saves pages
- **Storage list**: instead of one sheet per item, one packing list per storage
  location; individual locations can be ticked there. Sorting does not apply;
  the list is ordered by location, set and part.
- **Group identical sets** (storage list only): sets with the same name go
  under a single heading, and each part shows one picture instead of one per
  item. This saves a great deal of space — eight pages easily become two.

## When something gets stuck

Options → switch on *Show debug tab*. The new tab shows the log and lets you
copy it or save it as a text file. The filter at the top restricts it to
errors, warnings or notices.

The Info tab gives the storage locations of the settings and the database, as
well as the state of NFC.

## Deleting

The delete buttons for items and parts are hidden at first and can be switched
on in the Options.

What happens on deletion: the record disappears from the display but stays in
the database — merely marked, without its picture, and of the inspections the
latest one remains. The reason is the number: were it to become free again,
some other item would eventually carry the same identifier as on an old
printout.

You can look them up via **Export deleted (CSV)** in the menu — two files, one
for items, one for parts. The entry only appears with developer mode switched
on.

The database reset in the Options, by contrast, really does delete, with no
way back.