Our apps currently **use JSON files to know which GIFs they should display** in their GIF categories. The GIFs are all manually pre-selected a

All our GIFs currently come from GIPHY. 
NOTE: The below documentation does not take into account other GIF sources - a discussion and an update of this documentation should take place before using other sources of GIFs.

#### Current files used by the apps

At the moment,
- the iOS apps source their GIFs from the files in this Git folder: https://github.com/GhostWording/gw-config-apis/tree/master/data/common/giphycontent
- the Android apps source their GiFs from the files in this Git folder: https://github.com/GhostWording/gw-config-apis/tree/master/data/common/gifsnewformat

This is important because the files in each folder use a slightly different format. The folder used by the Android apps should soon be adopted by the iOS apps, so as to have only one source to update.

#### How to update a GIF category

There are over 30 GIF categories: **make sure to select the right sheet** when adding or deleting one. 
All the GIFs currently visible in the apps can be previewed in this Google Sheet: https://docs.google.com/spreadsheets/d/1r2zvV0za_u-Qg_fdP6xZbxXeTsDn3Wd5BcLenJTzWnA/edit?usp=sharing

Before adding or deleting a GIF on the Github file, you should update the Google Sheets, because it is the best way to identify and preview all the GIFs currently in the apps 

##### 1. Add or delete the GIF from Google Sheets

- to add, paste the link of your GIF in the last row of the B column
- to delete a GIF, delete the row altogether

##### 2. Paste changes into the JSON files

- if you're updating the JSON file for the Android apps (https://github.com/GhostWording/gw-config-apis/tree/master/data/common/gifsnewformat), you'll have to copy/paste the links in column F.
- if you're updating the JSON file for the iOS apps (https://github.com/GhostWording/gw-config-apis/tree/master/data/common/giphycontent), you'll have to copy/paste the links in column G.

To copy/paste, simply follow this guide

<img src="https://media.giphy.com/media/p4jcMzUaSzPHwkY1An/giphy.gif" width="720" height="381" />

You can find it here as well: [Alt Text](https://media.giphy.com/media/p4jcMzUaSzPHwkY1An/giphy.gif) 


That's it !
