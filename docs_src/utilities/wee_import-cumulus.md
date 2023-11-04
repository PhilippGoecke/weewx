!!! Warning
    Running WeeWX during a `wee_import` session can lead to abnormal termination of the import. If WeeWX must remain running (e.g., so that live data is not lost) run the `wee_import` session on another machine or to a second database and merge the in-use and second database once the import is complete.

`wee_import` can import observational data from the one or more Cumulus monthly log files. A Cumulus monthly log file records weather station observations for a single month. These files are accumulated over time and can be considered analogous to the WeeWX archive table. When `wee_import` imports data from the Cumulus monthly log files each log file is considered a 'period'. `wee_import` processes one period at a time in chronological order (oldest to newest) and provides import summary data on a per period basis.

## Mapping data to archive fields

A Cumulus monthly log file import will populate the WeeWX archive fields as follows:

* Provided data exists for each field in the Cumulus monthly logs, the following WeeWX archive fields will be directly populated by imported data:

    * `dateTime`
    * `barometer`
    * `dewpoint`
    * `heatindex`
    * `inHumidity`
    * `inTemp`
    * `outHumidity`
    * `outTemp`
    * `radiation`
    * `rain`
    * `rainRate`
    * `UV`
    * `windDir`
    * `windGust`
    * `windSpeed`
    * `windchill`

    !!! Note
        If a field in the Cumulus monthly log file has no data the corresponding WeeWX archive field will be set to `None/null`.

* The following WeeWX archive fields will be populated from other settings or configuration options:

    * `interval`
    * `usUnits`

* The following WeeWX archive fields will be populated with values derived from the imported data provided `calc_missing = True` is included in the `[Cumulus]` section of the import configuration file being used and the field exists in the in-use WeeWX archive table schema.

    * `altimeter`
    * `ET`
    * `pressure`

    !!! Note
        If `calc_missing = False` is included in the `[Cumulus]` section of the import configuration file being used then all of the above fields will be set to `None/null`. The default setting of the `calc_missing` option is `True`


## Step-by-step instructions

To import observations from one or more Cumulus monthly log files:

1. Ensure the Cumulus monthly log file(s) to be used for the import are located in a directory accessible by the machine that will run `wee_import`. For the purposes of the following examples, there are nine monthly logs files covering the period October 2016 to June 2017, inclusive, located in the `/var/tmp/cumulus` directory.

1. Make a backup of the WeeWX database in case the import should go awry.

1. Create an import configuration file. In this case we will make a copy of the example Cumulus import configuration file and save it as `cumulus.conf` in the `/var/tmp` directory:

    ```
    $ cp /home/weewx/util/import/cumulus-example.conf /var/tmp/cumulus.conf
    ```

1. Confirm the [`source`](../wee_import-config#import_config_source) option is set to Cumulus:

    ``` 
    source = Cumulus
    ```

1. Confirm that the following options in the `[Cumulus]` section are correctly set:

     * [directory](../wee_import-config#cumulus_directory). The full path to the directory containing the Cumulus monthly log files to be used as the source of the imported data.

     * [interval](../wee_import-config#cumulus_interval). Determines how the WeeWX interval field is derived.

     * [qc](../wee_import-config#cumulus_qc). Determines whether quality control checks are performed on the imported data.

     * [calc_missing](../wee_import-config#cumulus_calc_missing). Determines whether missing derived observations will be calculated from the imported data.

     * [separator](../wee_import-config#cumulus_separator). The date field separator used in the Cumulus monthly log files.

     * [delimiter](../wee_import-config#cumulus_delimiter). The field delimiter used in the Cumulus monthly log files.

     * [decimal](../wee_import-config#cumulus_decimal). The decimal point character used in the Cumulus monthly log files.

     * [ignore_invalid_data](../wee_import-config#cumulus_ignore_invalid_data). Determines whether invalid data in a source field is ignored or the import aborted.

     * [tranche](../wee_import-config#cumulus_tranche). The number of records written to the WeeWX database in each transaction.

     * [UV_sensor](../wee_import-config#cumulus_UV). Whether a UV sensor was installed when the source data was produced.

     * [solar_sensor](../wee_import-config#cumulus_solar). Whether a solar radiation sensor was installed when the source data was produced.

     * [[[Units]]](../wee_import-config#cumulus_units). Defines the units used in the Cumulus monthly log files.

1. When first importing data it is prudent to do a dry run import before any data is actually imported. A dry run import will perform all steps of the import without actually writing imported data to the WeeWX database. In addition, consideration should be given to any additional options to be used such as `--date`.
    
    To perform a dry run enter the following command:

    ```
    wee_import --import-config=/var/tmp/cumulus.conf --dry-run
    ```

    This will result in a short preamble with details on the data source, the destination of the imported data and some other details on how the data will be processed. The import will then be performed but no data will be written to the WeeWX database.
    
    The output should be similar to:
  
    ```
    Using WeeWX configuration file /home/weewx/weewx.conf
    Starting wee_import...
    Cumulus monthly log files in the '/var/tmp/cumulus' directory will be imported
    Using database binding 'wx_binding', which is bound to database 'weewx.sdb'
    Destination table 'archive' unit system is '0x01' (US).
    Missing derived observations will be calculated.
    This is a dry run, imported data will not be saved to archive.
    Starting dry run import ...
    Records covering multiple periods have been identified for import.
    Period 1 ...
    Unique records processed: 8858; Last timestamp: 2016-10-31 23:55:00 AEST (1477922100)
    Period 2 ...
    Unique records processed: 8636; Last timestamp: 2016-11-30 23:55:00 AEST (1480514100)
    Period 3 ...
    Unique records processed: 8925; Last timestamp: 2016-12-31 23:55:00 AEST (1483192500)
    Period 4 ...
    Unique records processed: 8908; Last timestamp: 2017-01-31 23:55:00 AEST (1485870900)
    Period 5 ...
    Unique records processed: 8029; Last timestamp: 2017-02-28 23:55:00 AEST (1488290100)
    Period 6 ...
    Unique records processed: 8744; Last timestamp: 2017-03-31 23:55:00 AEST (1490968500)
    Period 7 ...
    Unique records processed: 8489; Last timestamp: 2017-04-30 23:02:00 AEST (1493557320)
    Period 8 ...
    Unique records processed: 8754; Last timestamp: 2017-05-31 23:55:00 AEST (1496238900)
    Period 9 ...
    Unique records processed: 8470; Last timestamp: 2017-06-30 23:55:00 AEST (1498830900)
    Finished dry run import
    77813 records were processed and 77813 unique records would have been imported.
    ```
  
    !!! Note
        The nine periods correspond to the nine monthly log files used for this import.
  
    !!! Note
        Any periods for which no data could be obtained will be skipped. The lack of data may be due to a missing Cumulus monthly log file. A short explanatory note to this effect will be displayed against the period concerned and an entry included in the log.

1. Once the dry run results are satisfactory the data can be imported using the following command:

    ```
    wee_import --import-config=/var/tmp/cumulus.conf
    ```
  
    This will result in a preamble similar to that of a dry run. At the end of the preamble there will be a prompt:
  
    ```
    Using WeeWX configuration file /home/weewx/weewx.conf
    Starting wee_import...
    Cumulus monthly log files in the '/var/tmp/cumulus' directory will be imported
    Using database binding 'wx_binding', which is bound to database 'weewx.sdb'
    Destination table 'archive' unit system is '0x01' (US).
    Missing derived observations will be calculated.
    Starting import ...
    Records covering multiple periods have been identified for import.
    Period 1 ...
    Proceeding will save all imported records in the WeeWX archive.
    Are you sure you want to proceed (y/n)?
    ```
  
    If there is more than one Cumulus monthly log file then `wee_import` will provide summary information on a per period basis during the import. In addition, if the `--date` option is used then source data that falls outside the date or date range specified with the `--date` option is ignored. In such cases the preamble may look similar to:
  
    ```
    Using WeeWX configuration file /home/weewx/weewx.conf
    Starting wee_import...
    Cumulus monthly log files in the '/var/tmp/cumulus' directory will be imported
    Using database binding 'wx_binding', which is bound to database 'weewx.sdb'
    Destination table 'archive' unit system is '0x01' (US).
    Missing derived observations will be calculated.
    Starting import ...
    Records covering multiple periods have been identified for import.
    Period 1 ...
    Period 1 - no records identified for import.
    Period 2 ...
    Period 2 - no records identified for import.
    Period 3 ...
    Proceeding will save all imported records in the WeeWX archive.
    Are you sure you want to proceed (y/n)?
    ```

1. If the import parameters are acceptable enter `y` to proceed with the import or `n` to abort the import. If the import is confirmed, the source data will be imported, processed and saved in the WeeWX database. Information on the progress of the import will be displayed similar to the following:

    ```
    Unique records processed: 2305; Last timestamp: 2016-12-30 00:00:00 AEST (1483020000)
    ```
  
    Again if there is more than one Cumulus monthly log file and if the `--date` option is used the progress information may instead look similar to:
  
    ```
    Period 4 ...
    Unique records processed: 8908; Last timestamp: 2017-01-31 23:55:00 AEST (1485870900)
    Period 5 ...
    Unique records processed: 8029; Last timestamp: 2017-02-28 23:55:00 AEST (1488290100)
    Period 6 ...
    Unique records processed: 8744; Last timestamp: 2017-03-31 23:55:00 AEST (1490968500)
    ```
  
    !!! Note
        Any periods for which no data could be obtained will be skipped. The lack of data may be due to a missing Cumulus monthly log file. A short explanatory note to this effect will be displayed against the period concerned and an entry included in the log.
  
    The line commencing with `Unique records processed` should update as records are imported with progress information on number of records processed, number of unique records imported and the date time of the latest record processed. If the import spans multiple months (ie multiple monthly log files) then a new `Period` line is created for each month.
  
    Once the initial import is complete `wee_import` will, if requested, calculate any missing derived observations and rebuild the daily summaries. A brief summary should be displayed similar to the following:
  
    ```
    Calculating missing derived observations ...
    Processing record: 77782; Last record: 2017-06-30 00:00:00 AEST (1519826400)
    Recalculating daily summaries...
    Records processed: 77000; Last date: 2017-06-28 11:45:00 AEST (1519811100)
    Finished recalculating daily summaries
    Finished calculating missing derived observations
    ```
  
    When the import is complete a brief summary is displayed similar to the following:
  
    ```
    Finished import
    77813 records were processed and 77813 unique records imported in 106.96 seconds.
    Those records with a timestamp already in the archive will not have been
    imported. Confirm successful import in the WeeWX log file.
    ```

1. Whilst `wee_import` will advise of the number of records processed and the number of unique records found, `wee_import` does know how many, if any, of the imported records were successfully saved to the database. You should look carefully through the WeeWX log file covering the `wee_import` session and take note of any records that were not imported. The most common reason for imported records not being saved to the database is because a record with that timestamp already exists in the database, in such cases something similar to the following will be found in the log:

    ```
    Aug 22 14:38:28 stretch12 weewx[863]: manager: unable to add record 2018-09-04 04:20:00 AEST (1535998800) to database 'weewx.sdb': UNIQUE constraint failed: archive.dateTime
    ```
  
    In such cases take note of the timestamp of the record(s) concerned and make a decision about whether to delete the pre-existing record and re-import the record or retain the pre-existing record.