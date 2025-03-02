# Grants.gov Report Generator

## Description

This project generates US Government grant reports based on data provided by [grants.gov](https://www.grants.gov/). The user specifies the date-range of grants that they wish to include, then the program will generate a neatly formatted Microsoft Word document complete with a bookmarked table of contents. 

---

## Requirements

 * [Python 3.10](https://www.python.org/downloads/) or above
 * Ability to install Python packages with [Python pip](https://packaging.python.org/en/latest/tutorials/installing-packages/#requirements-for-installing-packages)  
 * All python packages in [requirements.txt](https://github.com/derek-chandler/grantsProject/blob/main/requirements.txt)
   * If you are using Windows, you can safely skip this step as the provided `GrantParser.bat` file will automatically install these packages.

To view the generated report properly, Microsoft Word is required.

---

## Usage

 * Windows:
    1. Install requirements as instructed above
    2. Execute the program by double clicking `GrantParser.bat`
 * Linux/macOS
    1. Install all requirements as instructed above
       * You may need to install tkinter on your system to be able to launch the GUI 
    2. Execute the program with `python GrantsParserXML.py`
    
 3. Choose a date range with the provided calendar GUI.
    * The default date range is from the past 7 days to the current date inclusive.
 4. Once a date range is selected, click **Confirm** and a report will be generated

When the program finishes, `GrantsReport_YYYY-MM-DD.docx` is generated and will be placed in the program's folder.

If you wish to generate another report *in the same day*, please rename or move the generated report out of the program's root directory

---

## * Initial Setup Proceess

[Windows]
Just run `GrantParser.bat` in the project folder.

or

1. Download the project folder (source code files).
2. Open cmd and change the directory to the folder.
3. Run `GrantParser.bat`: `$GrantParser.bat`

[MacOS]
1. Download the project folder (source code files).
2. Open Terminal and change the directory to the folder: `$cd Desktop/grantsParse`
3. Install the virtual environment package: `$pip install virtualenv`
4. Create a virtual environment named `venv` for this project: `$python3 -m venv venv`
5. Run the project virtual environment named `venv`: `$source venv/bin/activate`
6. Install the required python packages by using `requirements.txt`: `$pip install -r requirements.txt`
7. Run `GrantParserXML.py`: `$python GrantsParserXML.py`

---

## Changing the Template
  * GrantsParser has two possible templates: one with a Marshall University header and watermark and one with an Ops_Watch header and watermark
  * There are two provided templates in the programs root directory:
    * `Marshall template.docx` - this is the Marshall University template
    * `OpsWatch template.docx` - this is the Cornerstone Ops-Watch template
  * To change the template, search for "ChangeTemplate" in `GrantsParserXML`
  * There, you will see this line of code:
    * `doc = docx.Document(<DocumentName>)`
  * The filename given to doxc.Document(<DocumentName>) is an argument that will dictate teh template that will be used to generate our report
  * For example, to use the Marshall University template, the line of code below ChangeTemplate should look like this:
    * `doc = docx.Document("Marshall template.docx")`
  * To use the template for Cornerstone Ops-Watch, the line of code below ChangeTemplate should look like this:
    * `doc = docx.Document("OpsWatch template.docx")`
    * 
---

<br>
<br>

# Documentation


## Overview of driver `GrantsParserXML.py` 

MAIN Object

* Defines class Grant and init as well as all printed parameters in the grant report (IE Agency Code, Agency Name, etc)
* More information in *GrantsParserXML.py* section below.

MAIN Functions

* Defines the parameters printed in each individual grant report
* Defines grantDictionaryAdd that creates a dictionary using the distinctAgency as a key and the grants as values

Downloading

* Calls to the `GrantDownloader.py` file for it to begin downloading and extracting the most recent zipped XML.
* More information in *GrantDownloader.py* section below.

UI Begin

* Contains all UI Elements used for user to select date range of the grant report
* UI uses variable today and variable last_week to automatically select the default date range of the past 7 days
* Basic Settings defines the opening root of the tkinter UI panel as well as some settings such as the program title (root.title), UI Panel Icon (.ico for windows, .xbm for linux), and panel size (root.geometry)
* Line 227 - 231 defines the format of the UI frame, setting a "TOP" and "BOTTOM" of the UI panel in order to divide the placement of the ui elements
* my_toplabel sets a label value at the top of the UI panel while the .pack addition allows for the label to have padding and be placed within the top of the UI
* DateEntry fields set the two entry fiels with a popup calendar alongside the parameters of the calendar
* def grab_date collects the user's selected date from the DateEntry panels and sets an error popup if the user selects an improper date range (if the first date is AFTER the second date)
* def downloadxml is used within the multithreading in order to allow the XML download / unzip to run alongside the UI
* threading is used to run GrantDownloader and ensure both processes end before merging.
* my_button holds the parameters of the confirm button
* Line 279 ends the UI loop
* dateRangeOne and Two converts the date format of the DateEntry to a date.time object to a string using strftime

XML Parsing/Grant Generation

* Creates xmltree object to store the xml information
* Declare a list of agency names (agencyList) and a dictionary to store all grants (grantDictionary)
* Iterate through all grants in the xmltree structure, and create grants objects out of the grants with \<PostDate\> values between the given date range, inclusive. In this loop, we will also call tableOfConents method to add only unique distinctAgency names to agencyList and add any new grant to grantDictionary
* Once the loop ends, we will sort agencyList in order to use it as an ordered key call for our grantDictionary

<br>

## GrantDownloader.py

### Imported Default Libraries
 * os
 * sys
 * traceback
 * zipfile
 * http.client.RemoteDisconnected
 * time.sleep

### Imported External Libraries

 * requests
 * wget
 * bs4.BeautifulSoup
 * requests.exceptions.ConnectionError

### Functions

***cleanTmp***

 * Description
   * Deletes all files in the root directory of the script ending with `.tmp`
   * These files are generated by wget during download
   * If the download is stopped unexpectedly, these files remain on the system
 * Args
   * None

***cleanOldCache***

 * Description
   * Deletes `.zip` and `.xml` files in the `cache/` and `cache/extracted/` directories
   * Each XML dump and XML file range from about 40-60MB
   * Deleting these files saves significant storage space over time
 * Args
   * **currentfilename** : Current filename formatted like so: `GrantsDBExtractYYYYMMDD` without the .zip or .xml

***unzip_xml***

 * Description
   * Unzips a given `.zip` file to the `cache/extracted/` directory
   * The expected file is `.xml` since that is what is in the XML dumps provided on the Grants.gov website
 * Args
   * **file_path** : The path to the `.zip` file

***get***

 * Description
   * Creates `cache/` and `cache/extracted/` directory if they don't exist
   * Gets the latest XML dump URL using BeautifulSoup4 library
   * Checks if the latest dump is already downloaded
     * If the latest XML exists, return the filepath
     * If the latest ZIP exists but not XML, unzip and return filepath
     * If not downloaded, proceed
   * Downloads the XML dump zip file
   * Unzips the downloaded zip file
   * Returns the filepath of the XML file
 * Args
   * None

<br>

## GrantParserXML.py

### Imported Default Libraries

* datetime
* html
* os
* threading
* xml.etree.ElementTree
* tkinter
  * tkinter.BOTTOM
  * tkinter.LEFT
  * tkinter.RIGHT
  * tkinter.TOP
  * tkinter.Button
  * tkinter.Frame
  * tkinter.Label
  * tkinter.Tk
  * tkinter.messagebox

### Imported External Libraries

* docx
  * docx
  * docx.enum.text.WD_ALIGN_PARAGRAPH
  * docx.shared.Pt
* tkcalendar.DateEntry

### Imported Python Files

* word
* GrantDownloader

### Functions

***dateConversion***

 * Description
   * Creates date in MM/DD/YYYY format
 * Args
   * **date** : String of date in the form MMDDYYYY

***dateStringConversion***

  * Description
    * Creates date in Month DD, YYYY format
    * Uses monthList to select index of month referenced in MMDDYYYY
  * Args
    * **date** : String of date in the form MMDDYYYY

***addCommasAndDollarSign***

  * Description
    * Adds commas and a dollar sign to strings representing money values if the value needs it
  * Args
    * **amountStr** : string of numbers for a money value
  
***dateHierarchyForm***

  * Description
    * Converts date in MMDDYYY from to YYYYMMDD for purposes of comparison to other dates
  * Args
    * **date** : date to be converted to YYYYMMDD

***generateAgencyName***

  * Description
    * Takes a string representing the AgencyCode from a GrantsDBExtract XML file and derives the Agency Name from it
    * This is derived by taking the first substring preceding a '-' in the full string
    * Checks to see if this code corresponds to a key stored in the agencyDictionary. Returns the Agency value stored at the key if it's found, returns 'Other Agencies' if not
  * Args
    * **agencyCode** : string corresponding an AgencyCode from GrantsDBExtract XML file

***generateLink***

  * Description
    * Takes string corresponding to OpportunityID from a GrantsDBExtract XML file and creates a link to this grant on grants.gov by appending this string to the end of the common url
  * Args
    * **grantID** : string of common grants.gov url with OpportunityID

***wordLimiter***

  * Description
    * Takes a string and a number value to limit the number of words in a string delimited by white spaces
    * Returns string with the limit number of words, and ends with elipses if the string size is reduced
  * Args
    * **string** : string that needs to have a limited number of words
    * **limit** : number of words you wish the string to be below

***tableOfContents***

  * Description
    * takes string representing a distinctAgency and checks to see if it is in the list
    * if the string is in list a, do nothing. if the string is not, add the agency to the list
  * Args
    * **listA** : list of agencies to which you want to attempt adding a new agency
    * **agency** : candidate distinctAgecny string to be added to listA if it does not already exist

***getOpportunityInfo***

  * Description
    * Takes opportunity from the xml tree and returns the value corresponding to the attribute passed as an argument
    * shortens calls to specific values in the GrantsDBExtract XML document, reducing redundant code
  * Args
    * **opportunity** : branch of XML tree that represents a grant opportunity
    * **attribute** : string representing the name of an XML tag from which we want to return a value

### Class **Grant**


***\_\_init\_\_***

  * Description
    * creates a grants object to store the desired attributes
  * Args
    * **self** : this instantiation of a Grant object
    * **agencyCode** : AgencyCode attribute from GrantsDBExtract XML tree
    * **distinctAgency** : string generated using the first substring preceding a '-' character in agency code as a key to retrive the value from the set agencyDictionary
    * **agencyName** : AgencyName attribute from GrantsDBExtract XML tree
    * **opportunityTitle** : OpportunityTitle attribute from GrantsDBExtract XML tree
    * **postDate** : PostDate attribute from GrantsDBExtract XML tree
    * **dueDate** : CloseDate attribute from GrantsDBExtract XML tree
    * **numAwards** : ExpectedNumberOfAwards attribute from GrantsDBExtract XML tree
    * **totalFunding** : EstimatedTotalProgramFunding attribute from GrantsDBExtract XML tree
    * **awardCeiling** : AwardCeiling attribute from GrantsDBExtract XML tree
    * **awardFloor** : AwardFloor attribute from GrantsDBExtract XML tree
    * **oppNumber** : OpportunityNumber attribute from GrantsDBExtract XML tree
    * **description** : Description attribute from GrantsDBExtract XML tree
    * **eligApplicants** : AdditionalInformationOnEligibility attribute from GrantsDBExtract XML tree
    * **grantLink** : generated link using a common url for grants and the OpportunityID attribute from GrantsDBExtract XML tree
    * **contactInfo** : GrantorContactText attribute from GrantsDBExtract XML tree

***printGrant***

  * Description
    * Print stored attributes of a grant object inthe desired format
  * Args
    * **grant** : grant object that we would like to print

***grantDictionaryAdd***

  * Description
    * adds Grant object to a dictionary using the distinctAgency value as a key. if a Grant shares a key with a Grant object already stored in the grantDictionary, the new Grant is stored at the same key in a list with the previous Grant
  * Args
    * **grantDictionary** : the dictionary of Grant objects to which you would like to add a new Grant object
    * **grant** : Grant object you would like to add to the grantDictionary



## word.py

### Imported External Libraries
 * docx
   * docx
   * docx.Document
   * docx.enum.dml.MSO_THEME_COLOR_PACK
   * docx.enum.text.WD_ALIGN_PARAGRAPH
   * docx.oxml.xmlchemy.OxmlElement
   * docx.shared.Length
   * docx.shared.Pt
   * docx.text.paragraph.Paragraph

### Functions

***add_bookmark***

 * Description
   * Generates a bookmark into the document
 * Args
   * **paragraph** : a given paragraph object
   * **bookmark_text** : the text to place a bookmark at
   * **bookmark_name** : the internal name for the bookmark

***add_link***

 * Description
   * Generate a hyperlink that is linked to a bookmark
 * Args
   * **paragraph** : a given paragraph object
   * **link_to** : the internal name to link the bookmark to
   * **text** : the text to put in the paragraph
 * Optional args
   * **tool_tip** : a tooltip. set to `None` by default

***add_hyperlink***

 * Description
   * Adds a hyperlink that is connected to a website externally
 * Args
   * **paragraph** : a given paragraph object
   * **text** : the text to put in the paragraph
   * **url** : the website URL to add to the paragraph 

***insert_paragraph_after***

 * Description
   * Used to insert paragraphs at a certian index. 
   * It was necessary to manually create this method because the python DocX library did not have a built in method.
 * Args
   * **paragraph** : a given paragraph object
 * Optional args
   * **text** : text to put in the paragraph. set to `None` by default
   * **style** : paragraph styling options. set to `None` by default
