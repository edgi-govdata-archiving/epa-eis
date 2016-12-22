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

Before running you will need to get a valid jsessionid field and paste it into the script.  Without it, you will just get the same 20 results over and over because the paging isn't happening. Go to https://cdxnodengn.epa.gov/cdx-enepa-public/action/eis/search and do an empty search, and the URL of the results page will have a jsessionid field that will work.

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

### Unique results

`cut -d, -f1 eis-listing.csv | sort | uniq -c | sort -rn | head -5` will tell you if there are any duplicated lines, which may happen if more queries are made than necessary just to be on the safe side.

    2 83709
    2 83553
    2 83301
    2 83232
    2 82975

Tidy things up by doing this:

```sh
head -1 eis-listing.csv > tmp.csv
grep -v eis_id eis-listing.csv | sort -rn | uniq >> tmp.csv
mv tmp.csv eis-listing.csv
```

## 2: Get the metadata

This takes in a CSV (`../1-get-eis-ids/eis-listing.csv`) with a unique ID of each EIS page, and outputs JSON with the metadata and PDF links.

For example, if one of the links in the CSV document includes [this link](https://cdxnodengn.epa.gov/cdx-enepa-II/public/act
ion/eis/details?eisId=222951) with ID `222951`, then it will output this metadata JSON called `output/page-metadata-222951.json`:

```js
{
  "metaData": {
    "EIS Title": " Nexus Gas Transmission Project and Texas Eastern Appalachian Lease Project",
    "EIS Number": " 20160289",
    "Document Type": " Final",
    "Federal Register Date": " 2016-12-09 00:00:00.0",
    "EIS Comment Due/ Review Period Date": " 2017-01-09 00:00:00.0",
    "Amended Notice Date": "",
    "Amended Notice": "",
    "Supplemental Information": "",
    "Website": "",
    "EPA Comment Letter Date": "",
    "State or Territory": " MI - OH",
    "Lead Agency": " Federal Energy Regulatory Commission",
    "Contact Name": " Joanne Wachholder",
    "Contact Phone": " 202-502-8056",
    "Rating (if Draft EIS)": ""
  },
  "pdfLinks": [ {
    "pdf-link": "https://cdxnodengn.epa.gov#links",
    "pdf-filename": "EPA's PDF page"
  }, {
    "pdf-link": "https://cdxnodengn.epa.gov#links",
    "pdf-filename": "NEPAdatabasesupport"
  }, {
    "pdf-link": "https://cdxnodengn.epa.gov/cdx-enepa-II/public/action/eis/details;jsessionid=DACBBEC17397AF888EDC4C8470BB17A1?downloadAttachment=&attachmentId=223102",
    "pdf-filename": "1 Final Environmental Impact Statement.pdf"
  }, {
    "pdf-link": "https://cdxnodengn.epa.gov/cdx-enepa-II/public/action/eis/details;jsessionid=DACBBEC17397AF888EDC4C8470BB17A1?downloadAttachment=&attachmentId=223104",
    "pdf-filename": "2 Appendices A-D.pdf"
  }, {
    "pdf-link": "https://cdxnodengn.epa.gov/cdx-enepa-II/public/action/eis/details;jsessionid=DACBBEC17397AF888EDC4C8470BB17A1?downloadAttachment=&attachmentId=223106",
    "pdf-filename": "3 Appendices E1-E4.pdf"
  }, {
    "pdf-link": "https://cdxnodengn.epa.gov/cdx-enepa-II/public/action/eis/details;jsessionid=DACBBEC17397AF888EDC4C8470BB17A1?downloadAttachment=&attachmentId=223108",
    "pdf-filename": "4 Appendix E5 Part 1.pdf"
  }, {
    "pdf-link": "https://cdxnodengn.epa.gov/cdx-enepa-II/public/action/eis/details;jsessionid=DACBBEC17397AF888EDC4C8470BB17A1?downloadAttachment=&attachmentId=223110",
    "pdf-filename": "5 Appendix E5 Part 2.pdf"
  }, {
    "pdf-link": "https://cdxnodengn.epa.gov/cdx-enepa-II/public/action/eis/details;jsessionid=DACBBEC17397AF888EDC4C8470BB17A1?downloadAttachment=&attachmentId=223112",
    "pdf-filename": "6 Appendices F-Q.pdf"
  }, {
    "pdf-link": "https://cdxnodengn.epa.gov/cdx-enepa-II/public/action/eis/details;jsessionid=DACBBEC17397AF888EDC4C8470BB17A1?downloadAttachment=&attachmentId=223114",
    "pdf-filename": "7 Appendix R Part 1.pdf"
  }, {
    "pdf-link": "https://cdxnodengn.epa.gov/cdx-enepa-II/public/action/eis/details;jsessionid=DACBBEC17397AF888EDC4C8470BB17A1?downloadAttachment=&attachmentId=223116",
    "pdf-filename": "8 Appendix R Part 2.pdf"
  }, {
    "pdf-link": "https://cdxnodengn.epa.gov/cdx-enepa-II/public/action/eis/details;jsessionid=DACBBEC17397AF888EDC4C8470BB17A1?downloadAttachment=&attachmentId=223118",
    "pdf-filename": "9 Appendix R Part 3.pdf"
  }, {
    "pdf-link": "https://cdxnodengn.epa.gov/cdx-enepa-II/public/action/eis/details;jsessionid=DACBBEC17397AF888EDC4C8470BB17A1?downloadAttachment=&attachmentId=223120",
    "pdf-filename": "10 Appendix R Part 4.pdf"
  } ]
}

```

## 3: Archive as WARC

This is a Python script that will parse through the CSV from stage 1 using `wget` to get the EIS files and package them in WARC format and download any documents associated with the EIS into a zip format.

`createWarc.py` invokes `wget` command as a subprocess in the script to package the response from the supplied URL into WARC. The PDF links present in the HTML page cannot be preserved in the HTML, therefore they are downloaded separately.

## Installation and preparation

```sh
git clone git@github.com:guerrilla-archiving/epa-eis.git
cd epa-eis
gem install nokogiri
cd 2-scrape-metadata
npm install
npm installcsvtojson
cd ..
```

(Depending on your system you may need to run some of these commands with the prefix `sudo`.)

## Usage

```sh
cd epa-eis/1-get-eis-ids/
# Make sure you pasted a fresh jsessionid into the script
ruby scrape-eis.rb > eis-listing.csv
cd ../2-scrape-metadata/
npm start
cd ../3-make-warcs/
python createWarc.py
```

## Dependencies

Each stage uses a different language, but evverything is available for all operating platforms.

For stage 1:

* Ruby
* [Nokogiri](http://www.nokogiri.org/)

For stage 2:

* [Node.js](https://github.com/nodejs/node)
* [npm](https://www.npmjs.com/)

For stage 3:

* [wget](https://www.gnu.org/software/wget/)
* [Python](https://www.python.org/) 2.7
