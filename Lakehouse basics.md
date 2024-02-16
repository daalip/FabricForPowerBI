# Lakehouse Basics

## Create a Workspace

Navigate to Fabric Home through Power BI - https://app.powerbi.com

Select Workspaces from the left rail

Select New workspace - Use the name "Fabric for Power BI Professionals"

Open the advanced tab and make sure that the License mode is Trial, Premium, or Fabric. If not, set it to one of these

## Create a Lakehouse

Select the "New" button in the ribbon. Notice that Lakehouse is not one of the options. You have 2 options to create a Lakehouse

1. Select "More Options" and select the Lakehouse tile from the Data Engineering section or
2. Change your persona to Data Engineering using the persona selector at the bottom left of the screen

Give the Lakehouse a name - we suggest "My_Lakehouse". Note that the name can NOT include spaces

Note the two containers, Files, and Tables. Note that they are empty


## Add File content

Download all of the CSVs located [here](https://github.com/jasonhimmelstein/FabricForPowerBI/tree/main/Recent%20Thermostat%20Data) to your local device.

From your Lakehouse, click the ellipsis to the right of the Files container, and select "New subfolder". Name it "Recent thermostat files"

Using the ellipsis to the right of your new folder, select "Upload". and upload the CSV for August that you downloaded above

Click on the file to view the content

Hover over the row, and select the ellipsis that appears. Select Properties.

Note the URL that appears. This could be used with the ADLG2 connector in Power BI Desktop to build a report against this data, but we will use another mechanism

## Move CSV to Delta tables

Hover over the CSV file, select the ellipsis, and click on "Load to Tables".

Select New table and name your table "thermostat_readings". This will take a moment, and will load the data from the CSV into the Tables endpoint where it will be stored in parquet files using Delta to allow for transactional access

Open the Tables node. If it is empty, select refresh from the ribbon on the top of the page. Note the new table there with the small triangle icon. That icon indicates that the table is in Delta format.

Click on the new table and observe the preview (1000 rows).

## Create Power BI report from the table (PBI Desktop)

Start Power BI Desktop

Select the Get data button from the ribbon

Choose the Microsoft Fabric category

Choose Lakehouse, then select "My_Lakehouse" from the list. Notice that you did not need to authenticate, the logged in user's credentials were passed in

Click "Connect"

Notice that Power Query was not loaded, and the table created above appears automatically

Drag Outdoor Temperature and Time onto a line chart on the canvas. Notice that the sum of temperature is used, but we cannot change the aggregation here. We are connected to a remote semantic model as noted in the status bar at the bottom**.** Also note the fact that there is no date hierarchy.

Close Power BI Desktop without saving

## Create a View for time intelligence

Open the "Fabric for Power BI Professionals" workspace in the service, and then open the "My_Lakehouse" Lakehouse

From the upper right hand corner, change the "Lakehouse" dropdown to "SQL analytics endpoint"

(Alternatively, you could select the lakehouse's SQL analytics endpoint from the workspace)

Select the "New visual query" button from the ribbon.

Drag the "thermostat_readings" table onto the canvas

Select the "Manage columns" button from the ribbon, and click on Choose columns. Deselect all columns except Date, Time, Outdoor_Temp, and the columns sta.ting with Cool_Stage and Heat_Stage

Today, we can't add columns using the visual editor, so click on "View SQL" from the ribbon, then select "Edit SQL script" at the bottom.

Add lines to extract year, month and day into separate columns. Also, cast the Time column to Time instead of DateTime. When complete, the query should appear as follows:

```Sql
select [Date],
    Cast([Time] as Time) as [Time],
    [Outdoor_Temp],
    [Cool_Stage_1],
    [Cool_Stage_2],
    [Heat_Stage_1],
    [Heat_Stage_2],
    YEAR([Date]) as [Year],
    MONTH([Date]) as MonthNumber,
    DATENAME(Month,[Date]) as Month,
    DAY([Date]) as [Day]
from [My_Lakehouse].[dbo].[thermostat_readings] as [$Table]
```

Run the query to ensure that it works. When satisfied, select the query text and click the "Save as view" button. Name your view "vwThermostatReadings"

## Edit the Semantic Model in the service

Select the View created above to preview the data

Select the "New Measure" button from the ribbon. This allows you to create a new DAX measure from the SQL user interface

Create the following measure:

```PlainText
Average Temperature = AVERAGE(vwThermostatReadings[Outdoor_Temp])
```

Note that the measure refers to the view created above.

Select the Model button at the bottom of the left pane

Drag the "vwThermostatReadings" table to the left side of the screen. Scroll down and note that "Average Temperature" has been added to the table.

Right click on one of the other tables on the canvas, and select "Hide in report view". Repeat for the other tables, but not the view. 

Select "Month" from the vwThermostatReadings table. In the properties pane on the right, open the advanced section, and choose MonthNumber in the "Sort by column" dropdown. This will allow the month name to be sorted chronologically.

## Create Power BI report in the service

From the ribbon in the model view or the data view, select "New report"

Minimize the Filters pane

Add a Line and Clustered column chart visual to the canvas

Open the vwThermostatReadings table

Add Cool_Stage_1, Cool_Stage_2, Heat_Stage_1, Heat_Stage_2 to the Column y-axis

Add Average Temperature to the Line y-axis

Add Year, Month, and Day to the X axis (in order). Note that while it is not currently possible to create a hierarchy in the model, this will have the same effect

Save the report, and name it "Thermostat report 1"

In this lab, you have ingested data, transformed it, modified the default model and bult a report, all without having to use Power BI Desktop.