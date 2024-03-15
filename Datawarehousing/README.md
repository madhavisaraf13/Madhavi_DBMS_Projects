
Technologies used, Concepts covered: AWS RDS/ local MYSQL database for construction of database, R, Star/snowflake schema, Fact table, OLTP/OLAP, datawarehouse. XML Parsing, DTD for XML validation, XPath

Workflow:
Extract data from an XML document (31 MB with ~1 million lines) and then transform and store the data relationally in a SQLite database which represents a "transactional" database. 
Then I am extracting the  data from the transactional database and create an "analytical" database/loading it into a datawarehouse using a star schema in MySQL. 
Finally, querying the facts from the MySQL analytical database / datawarehouse which will require that you connect to two different databases simultaneously.


In this project the ETL pipeline is created which can be elaborated as follows:

- Step 1 (Extraction & Transformation) : Data extraction and preprocessing/cleaning : In this step we are parsing the XML document and extracting the necessary information and performing transformation by data preprocessing to make the data as consistent as possible as described at the end of this document.

- Step 2 Load the data(Loading) : Loading the data into the MYSQL database and loading the data warehouse from the MYSQL database

- Step 3 : Querying the database to extract the necessary facts/information.

The AnalyzeDataReport html shows the report of the analysis after querying the datawarehouse.

Steps taken for data cleaning for Step 1:
 - Two articles are considered to be same/duplicate if their pmid and title are same.
    - Two jouranl rows in the journal table are considered to be same if their issn,issnType,jtitle,isoAbbr,journalissue(volume,issue,date),citedMedium
    -  Two authors are considered to be duplicate if their fname,lname,initials,suffix,affiliation are same.
    - Parsing dates:
        -1. If there is a combination such as Season tage and year tag is present. Then based on the season, the month is initilalized using the hash as below:
            hashes used for parsing season
            season.map <- hash()
            season.map[["summer"]] <- "jun"
            season.map[["winter"]] <- "dec"
            season.map[["fall"]] <- "sep"
            season.map[["spring"]] <- "mar"
        -2. Different kinds of patterns for Medline date have been identified and parsing has been done to extract information from all the possible
            varying patterns present in the file for MedlineDate tag.If the years, months or days are hyphenated, only the first part before the hyphen for Year/Month/Day is considered. In case, Year, Month or Day is not present. Then default values are used. Year -> '1111', Month -> 'dec', Day -> '15'.

            Possible Medline date patterns:
            <MedlineDate>1975 Jul-Aug</MedlineDate>
            <MedlineDate>1975 Dec-1976 Jan</MedlineDate>
            <MedlineDate>1975-1976</MedlineDate>
            <MedlineDate>1975 Aug 15-31</MedlineDate>
            <MedlineDate>1976 Aug 28-Sep 4</MedlineDate>
            <MedlineDate>1976 Dec 23-30</MedlineDate>
            <MedlineDate>1976-1977 Winter</MedlineDate>
            <MedlineDate>1976 Sep 30-Oct 2</MedlineDate>
            <MedlineDate>1977-1978 Fall-Winter</MedlineDate>

        - 3. Even after parsing MedlineDate/normal date, the exact year, month, day are not found. Then default values are initialized. Year -> '1111', Month -> 'dec',  Day -> '15'
