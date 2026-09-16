# warehouse-slotting-tool
Power BI dashboard for analyzing picking activity and sales history in the warehouse to support optimized inventory planning decisions

## Business Problem
When planning our warehouse floor, we must balance high and low-demand articles in an efficient way. This means high-demand articles should be as accessible as possible, while low-demand articles should be given low-priority locations. Given that our warehouse has thousands of articles and only a fraction of floor and shelf slots to accommodate them, we must be careful about the way we plan our warehouse articles. 

Our historical sales and order picking data are difficult to analyze in their raw form. When we are making inventory planning decisions, we typically find ourselves digging through multiple raw reports and planning without a complete view of warehouse activity. There is no simple way for us to compare the performance of every article or to see which articles have locations that may not fit their level of demand.

A poorly planned warehouse can negatively affect our order picking productivity and create frustration for coworkers, who may have to consistently take longer to pick the same article because of its location. These locations could either be too far from our main picking area or stored in elevated racking that requires a forklift to retrieve. Over time, inefficient planning creates unnecessary travel time and equipment use during the picking process.

## Solution
To provide a complete and insightful view of warehouse sales activity, I developed a Power BI dashboard that combines historical picking data with article and warehouse location data. This tool provides three different pages for analyzing article demand and identifying articles that are potential candidates for relocation.

## Article Picking Analysis

<img width="1316" height="737" alt="image" src="https://github.com/user-attachments/assets/041df670-3324-4814-b42f-e874723cafa6" />

The **Article Picking Analysis** page provides the picking activity of every warehouse article. Users can filter articles by location type (floor or shelf), aisle, product area, product division, pallet size, and a range of dates to identify both high and low-demand articles throughout the warehouse.

*(Dates have been erased from the date slicer to protect historical sales data)*

## PALLET Article Performance

<img width="1672" height="941" alt="pallet picking analysis" src="https://github.com/user-attachments/assets/01ad767c-3ba4-4c6e-b6d9-c3aafce0c296" />

*(For reference, a **PALLET** article is an article that lives in elevated racking and does not have a home location on our warehouse floor)*

The **PALLET Article Performance** page helps us identify which articles stored in elevated racking are picked most frequently. This tool can be used alongside the **Article Picking Analysis** page to identify articles stored on our floor with low picking demand, as these are the strongest candidates for a location swap. By relocating high-demand articles to the floor and low-demand articles to the elevated racking, we make frequently picked articles more accessible to our coworkers, reducing unnecessary forklift use and improving our overall picking efficiency.

The vertical bar graph in the top right allows us to view PALLET picks week-over-week to determine if PALLET picking demand is consistent over time or concentrated within certain weeks. The horizontal bar graph on the bottom right displays the top 10 PALLET articles by pick count, allowing us to quickly identify which articles are generating the most PALLET picks.

## Picking Heat Map and Tooltip

<img width="727" height="727" alt="image" src="https://github.com/user-attachments/assets/cb99fe43-690c-41fe-8b14-2813e4370cc9" />

The **Picking Heat Map** provides a visual representation of picking activity throughout our warehouse by aisle and bin location. Conditional formatting is used to highlight high and low activity locations, while the "IKEA" and "EURO" identifiers at the top of the visual allow us to distinguish which aisles are "IKEA-length" (long) and "Euro-length" (short) locations.

Each heat map cell also includes a custom tooltip that provides additional details about the articles assigned to each aisle and bin location. Hovering over a cell displays the articles stored in that location, allowing the user to investigate a spot without leaving the heat map.

<img width="765" height="727" alt="image" src="https://github.com/user-attachments/assets/8b247bde-2d3e-4531-ba51-442cea5e97fc" />

In the example above, we see that there are two articles located in Aisle 30, Bin 65. These two articles combine for a total of 162 picks during a set time period.

## Data Ingestion

This dashboard pulls data from three separate reports that provide information about the articles, such as picking/sales history, article name, article number, and assigned location in the warehouse. These reports are automatically exported as Excel files to SharePoint each day, replacing the previous day's files. Power BI connects to these folders and receives the updated data each morning. Power BI connects to these SharePoint folders as its data sources, allowing the semantic model to receive the updated data.

*(Historical sales data, forecasting, and data source table names have been redacted for data privacy reasons)*

### Source Data

The three reports mentioned above are briefly described below. For confidentiality, the internal report names have been replaced with descriptive names throughout this project.

| Source | Purpose |
| --- | --- |
| **Article Dimensions** | Provides article dimensions and other physical product attributes used for pallet-size classification. |
| **Sales Space Optimization** | Provides current article and warehouse-location information used to identify where articles are stored. |
| **Picking Reports** | Historical order-picking data used to calculate article demand and picking activity by warehouse location. |

## Data Transformation

Before building the data model, I used Power Query to clean up and standardize the source data. Just as I did in the **warehouse-location-optimization** repository, I needed to ensure consistent formatting for article numbers and location IDs (`SLID`'s).

Our article numbers use eight-digit numerical identifiers. Some of my source reports omitted leading zeros for this identifier, so I used `Text.PadStart` to fix this.

```DAX
= Table.AddColumn(Table1_Table, "ARTNO_fixed", each Text.PadStart(Text.From([ARTNO]), 8, "0"))
```

Then, because of how the **Picking Reports** data is retrieved, the report can sometimes generate duplicate rows. To find these duplicate rows, I used the **Remove Duplicates** function in Power Query based on specific values in each row that can identify a unique picking record.

```DAX
= Table.Distinct(#"Changed Type", {"Order No", "Order Type", "Article No", "Pick Area", "User Picking"})
```

Our warehouse `SLID` values contain six digits representing the aisle, bin, and level of each location. I separated these values into individual columns so warehouse locations could be analyzed within the **Picking Heat Map**.

```DAX
Aisle = Text.Start(Text.From([SLID]), 2)

Bin = Text.Middle(Text.From([SLID]), 2, 2)

Level = Text.End(Text.From([SLID]), 2)
```

## Data Modeling

After cleaning and standardizing the data, I was able to create relationships between each table. Below is the model view of the Power BI that displays the relationships of each table:

<img width="1567" height="1004" alt="bin planning tool relationship model " src="https://github.com/user-attachments/assets/95982212-8543-4530-bc00-d294f26e5a98" />

| Side One                    | Relationship Key                 | Side Two                       | Cardinality       |
| --------------------------- | -------------------              | --------------------------     | ----------------- |
| **SM2 Articles**            | `ArticleNo`                      | **Sales Space Optimization**   | One-to-Many (1:*) |
| **SM2 Articles**            | `ArticleNo`                      | **Picking Reports**            | One-to-Many (1:*) |
| **Calendar**                | `Date` ↔ `Date Orderline Picked` | **Picking Reports**            | One-to-Many (1:*) |
| **Full Serve Locations**    | `SLID`                           | **Picking Reports**            | One-to-Many (1:*) |
| **Full Serve Locations**    | `SLID`                           | **Sales Space Optimization**   | One-to-Many (1:*) |
| **SM2 Articles**            | `ArticleNo`                      | **Article Dimensions**         | One-to-One (1:1)  |

The **SM2 Articles** table serves as the primary article-level table in the model. It connects each unique article to the **Picking Reports**, **Sales Space Optimization**, and **Article Dimensions** tables. These relationships allow article details like location information, physical pallet dimensions, and historical picking activity to be analyzed together.

The **Full Serve Locations** table contains one row for each warehouse location and connects to both **Sales Space Optimization** and **Picking Reports** through the `SLID` field. This allows updated warehouse location information to be compared with the picking data and provides the aisle and bin numbers used to build the **Picking Heat Map** tab.

Finally, a **Calendar** table connects to the **Picking Reports** table using the date each article was picked, allowing picking data to be filtered and analyzed across any specific range of dates.

## DAX & Analytical Layer

After cleaning and modeling the source data, I used DAX to create calculated tables, calculated columns, and measures needed for the dashboard. These calculations transform the raw data into article-level information that is used throughout the dashboard.

### SM2 Articles Calculated Table

The **SM2 Articles** table seen in the relationship model is a table I created to have one summarized record for each article stored in our warehouse. The source **Sales Optimization** report can contain multiple rows for the same article when it is stored in multiple locations, so I used `SUMMARIZE` to create this article-level table.

```DAX
SM2 Articles = 
SUMMARIZE(
    FILTER(
        'Sales Space Optimization',
        NOT(ISBLANK('Sales Space Optimization'[ArticleNo]))
    ),
    'Sales Space Optimization'[ArticleNo],

    "SLID",
        CONCATENATEX(
            VALUES('Sales Space Optimization'[SLID]),
            'Sales Space Optimization'[SLID],
            ", ",
            'Sales Space Optimization'[SLID],
            ASC
        ),

    "Article Name",
        MIN('Sales Space Optimization'[ARTNAME_UNICODE]),

    "Product Division",
        MIN('Sales Space Optimization'[Product Division]),

    "Product Area",
        MIN('Sales Space Optimization'[Product Area])
)
```

`FILTER` removes records without a valid article number. `SUMMARIZE` then groups the remaining data by article number and `CONCATENATEX` combines all of the article's assigned locations (if there is more than one) into a single value that separates each location with commas.

`MIN` is used for identifiers such as **Article Name,** **Product Division,** and **Product Area** to return a single value for each summarized article.

The resulting table serves as the main article-level table in the data model and is used throughout the dashboard for analysis and filtering at the article level.

### Location Classification Type

For the **Article Picking Analysis** page, I wanted users to be able to filter articles stored either in floor or shelf locations. Since there is no column in any of the source data that directly identifies this attribute, I created a `Location Type` calculated column to assign a location type to each article.

```DAX
Location Type = 
IF(
    RIGHT(FORMAT('SM2 Articles'[SLID], "000000"), 2) = "00",
    "Floor",
    "Shelf"
)
```

Our warehouse locations use a six-digit numerical `SLID`, where the final two digits represent a storage level. A level of `00` represents a floor location, while all other values represent a shelf location.

`FORMAT` ensures the `SLID` contains six digits, while `RIGHT` extracts the final two characters. The `IF` statement then classifies each article as either **Floor** or **Shelf**.

### Pallet Size Classification

Physical product dimensions are also important when we are evaluating our current warehouse layout for potential article swaps. Some pallets require longer warehouse locations, which are far more scarce in our current layout. I created a calculated column that looks up the gross pallet length for each article from the **Article Dimensions** table.

```DAX
Pallet Size = 
VAR PalletLength =
    LOOKUPVALUE(
        'Article Dimensions'[UL_LENGTH_GROSS_CM],
        'Article Dimensions'[ARTNO_fixed],
        'SM2 Articles'[ArticleNo]
    )
RETURN
SWITCH(
    TRUE(),
    ISBLANK(PalletLength), BLANK(),
    PalletLength > 145, "IKEA",
    "EURO"
)
```

`LOOKUPVALUE` matches the current article to its dimensions listed in the corresponding report, while the `PalletLength` variable stores the retrieved value to be used in the formula. This variable is then evaluated in the `RETURN` expression to return the appropriate size classification.

Any article longer than 145cm would need to be classified as an **IKEA** length pallet, while anything at or below 145cm is classified as a **EURO** pallet. Any article without dimensional data would return a blank.

This provides additional information when we are planning, as we need to take into account more than just an article's demand when bin planning.

### Dynamic Calendar Table

To analyze picking activity over time, I created a dedicated Calendar table based on all dates available in the **Picking Reports** table.

```DAX
Calendar = 
CALENDAR(
    MIN('Picking Reports'[Date Orderline Picked]),
    MAX('Picking Reports'[Date Orderline Picked])
)
```

`MIN` and `MAX` identify the earliest and latest dates in the **Picking Report** data, while `CALENDAR` generates a table of dates covering that entire period.

This Calendar table is related to the **Picking Reports** table and allows the dashboard to filter picking activity by a date or range of dates and also analyze PALLET picking activity week-over-week.

### Picking Activity Measures

There are two measures used throughout the dashboard to evaluate article demand by retrieving historical picking data. These two measures are **Pick Count** and **Picked QTY Total**. Pick Count measures how frequently an article appears in the picking data, while Picked QTY Total measures the total quantity picked.

### Pick Count

```DAX
Pick Count = 
COALESCE(
    COUNTROWS('Picking Reports'),
    0
)
```

`COUNTROWS` counts the number of rows within the context of the current filter. Because Power BI automatically applies the filters from the selected article, location, date, and other selections, this measure can be used in multiple tabs of the dashboard.

`COALESCE` converts blank results to 0, which is important when identifying articles that haven't experienced any picking activity during the selected period.

### Picked Qty Total

```DAX
Picked QTY Total = 
COALESCE(
    SUM('Picking Reports'[Picked Qty]),
    0
)
```

Using both measures together provides a bigger picture when evaluating article demand, as one article might generate many picks for small quantities, or fewer picks for larger quantities. 

## Skills Demonstrated

- Power BI dashboard development and interactive report design
- Power Query data cleaning and transformation
- Data ingestion and duplicate removal
- Relational data modeling and dimension tables
- DAX calculated tables, calculated columns, and measures
- Dynamic calendar/date modeling
- Warehouse location parsing and classification
- Conditional formatting and heat-map visualization
- Report-page tooltips and cross-filtering
- Operational data analysis and warehouse slotting



