# EPA Environmental Impact Statement Collection

One of the [goals set out for the technical group](https://github.com/guerrilla-archiving/eot-sprint-toolkit/blob/master/Tech-Group-Goals.org) at the Guerrilla Archiving Event (at the University of Toronto, 16 December 2016) was:

> # Scraper for the EPA EIS Collection
>
> Environmental Impact Statements or EIS are a crucial tool in environmental regulation.  [[https://cdxnodengn.epa.gov/cdx-enepa-public/action/eis/search][They are accessible here]], and leaving all search criteria blank returns a full listing. Documents are accessible at landing pages two clicks down, which [[https://cdxnodengn.epa.gov/cdx-enepa-II/public/action/eis/details?eisId=223815][look like this]].  These landing pages contain metadata which we should scrape.
>
> ## Deliverables
>
> - *Code:* write a scraper/crawler that can extract metadata form text & PDF files & save them to a simple DB for future researchers.
>
> ## Priority
>
> EIS are a high-value document source, but it looks like they will also be retrieved by the official IA crawl. So this is *not* time-sensitive.

## Outline

Our process is built in three steps:

1. Scraping all EIS links and outputing a CSV with unique IDs

2. Scrape each page for metadata (EIS Number, PDF links, etc)

3. Process to convert all links and PDFs to WARC format

## 1: Get ID numbers

A full listing of all Environmental Impact Statements is available at https://cdxnodengn.epa.gov/cdx-enepa-public/action/eis/search by doing an empty search. Unfortunately the CSV export there isn't useful because it doesn't include the ID numbers of the statements, so we need to go through all the pages of the results and scrape them to get the information out.

`scrape-eis.rb` does this.

Usage: `ruby scrape-eis.rb > eis-listing.csv`

The CSV file has these columns, all pulled from the database:

* eis_id
* title
* document
* epa_comment_letter_date
* federal_register_date
* agency
* state

The important field is the `eis_id`.

For example, [https://cdxnodengn.epa.gov/cdx-enepa-II/public/action/eis/details?eisId=219321](219321), "Effects of Oil and Gas Activities in the Arctic Ocean."

At the bottom of the page there are links to several EIS documents and a comment letter.  On the search results page there are links to ZIP files for each.

* Documents: https://cdxnodengn.epa.gov/cdx-enepa-II/public/action/eis/details/downloadEisDocuments?eisId=219321
* Comments: https://cdxnodengn.epa.gov/cdx-enepa-II/public/action/eis/details/downloadCommentLetters?eisId=219321

Knowing the eis_id and those URL patterns means it's easy to download all the documents and comments (but make sure to rename the downloaded ZIP files, because they do not have unique names).

## 2: Get the metadata

[To be done.]

## 3: Archive as WARC

This directory has a Python script that will parse through the CSV from stage 1 use `wget` to get the EIS files and package them in WARC format and download any documents associated with the EIS into a zip format.

`createWarc.py` invokes `wget` command as a subprocess in the script to package the response from the supplied URL into WARC. The PDF links present in the HTML page cannot be preserved in the HTML, therefore they are downloaded separately.


# Dependencies

For stage 1:

* Ruby
* [Nokogiri](http://www.nokogiri.org/): `gem install nokogiri`

For stage 2:

For stage 3:

1. `wget` (available for all operating systems, see https://www.gnu.org/software/wget/)
2. Python 2.7

## Usage

Clone this repo then just simply run the python script `python createWarc.py`

Supplied is the csv, if you would like to customize this python or expand and modify the script to a take in generic datasets please modify the csv, base urls accordingly.
