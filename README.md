# Automotive-Revenue-and-Insights-Dashboard
This project presents an Automotive Revenue & Customer Insights Dashboard developed using Power BI. The analysis focuses on vehicle purchases, customer demographics, and revenue trends across multiple automotive brands. Power BI was used not only for visualization but also for all data preparation and transformation.

The data cleaning process was performed in Power Query, where inconsistent formats, invalid values, and missing fields were corrected to ensure a reliable dataset. New time-based fields—Month and Year—were extracted from the main date column to enable deeper trend analysis and interactive filtering. Additionally, a new calculated column for Revenue was created by multiplying the unit price by the number of items purchased, providing a clear view of financial performance across categories.

Using these prepared fields, the dashboard highlights total revenue, customer age groups, monthly purchase behavior, gender distribution, and model-level performance patterns. Interactive filters allow users to explore insights across different customer segments, years, and car brands.

Overall, this dashboard offers a clean, well-structured, and insight-driven view of sales performance, built end-to-end within Power BI—from data cleaning to modeling and final reporting.

#POWER QUERY M CODE(Data cleaning + Month, Year + Revenue column)

let
    // Load source file
    Source = Excel.Workbook(File.Contents("CarSalesData.xlsx"), null, true),
    Sales_Sheet = Source{[Item="Sales", Kind="Sheet"]}[Data],

   // Promote first row to headers
    PromotedHeaders = Table.PromoteHeaders(Sales_Sheet, [PromoteAllScalars=true]),

// Clean text fields: trim spaces and standardize
    CleanText = Table.TransformColumns(
        PromotedHeaders,
        {
            {"CustomerName", Text.Trim, type text},
            {"Gender", Text.Proper, type text},
            {"Brand", Text.Proper, type text},
            {"Model", Text.Proper, type text}
        }
    ),
    // Change data types
    ChangeTypes = Table.TransformColumnTypes(
        CleanText,
        {
            {"CustomerID", Int64.Type},
            {"Age", Int64.Type},
            {"Purchase Date", type date},
            {"Price", type number},
            {"Purchased", Int64.Type}
        }
    ),

   // Remove rows with missing key values
    RemovedNulls = Table.SelectRows(
        ChangeTypes,
        each [Purchase Date] <> null and [Price] <> null and [Purchased] <> null
    ),
    // Create Revenue Column (Price × Purchased)
    AddRevenue = Table.AddColumn(
        RemovedNulls,
        "Revenue",
        each [Price] * [Purchased],
        type number
    ),

   // Extract Year
    AddYear = Table.AddColumn(
        AddRevenue,
        "Year",
        each Date.Year([Purchase Date]),
        Int64.Type
    ),

// Extract Month Name
    AddMonth = Table.AddColumn(
        AddYear,
        "Month",
        each Date.MonthName([Purchase Date]),
        type text
    ),

  // Final sorted data
    SortedRows = Table.Sort(AddMonth, {{"Purchase Date", Order.Ascending}})
in
    SortedRows
