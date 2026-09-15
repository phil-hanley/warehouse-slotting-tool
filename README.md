# warehouse-slotting-tool
Power BI dashboard for analyzing picking activity and sales history in the warehouse to support optimized inventory planning decisions

## Business Problem
When planning our warehouse floor, we must balance high and low-demand articles in an efficient way. This means high-demand articles should be as accessible as possible, while low-demand articles should be given low-priority locations. Given that our warehouse has thousands of articles and only a fraction of floor and shelf slots to fit these, we must be careful about the way we plan our warehouse articles. 

Our historical sales and order picking data are difficult to analyze in their raw form. When we are making inventory planning decisions, we typically find ourselves digging through multiple raw reports and planning without a complete view of warehouse activity. There is no simple way for us to compare the performance of every article or to see which articles have locations that may not fit their level of demand.

A poorly planned warehouse can negatively affect our order picking productivity and create frustration for coworkers, who may have to consistently take longer to pick the same article because of its location. These locations could either be too far from our main picking area or stored in elevated racking that requires a forklift to retrieve. Over time, inefficient planning creates unnecessary travel time and equipment use during the picking process.

## Solution
To provide a complete and insightful view of warehouse sales activity, I developed a Power BI dashboard that combines historical picking data with article and warehouse location data. This tool provides three different pages for analyzing article demand and identifying articles that are potential candidates for relocation.

## Article Picking Analysis

<img width="1316" height="737" alt="image" src="https://github.com/user-attachments/assets/041df670-3324-4814-b42f-e874723cafa6" />

The **Article Picking Analysis** page provides the picking activity of every warehouse article. Users can filter articles by location type (floor or shelf), aisle, product area, product division, pallet size, and a range of dates to identify both high and low-demand articles throughout the warehouse.

*(Dates have been erased from the date slicer to protect historical sales data)*

## PALLET article performance

<img width="1672" height="941" alt="pallet picking analysis" src="https://github.com/user-attachments/assets/01ad767c-3ba4-4c6e-b6d9-c3aafce0c296" />

The **PALLET Article Performance** page helps us identify which articles stored in elevated racking are picked most frequently. This tool can be used alongside the **Article Picking Analysis** page to identify articles stored on our floor with low picking demand, as these are the strongest candidates for a location swap. By relocating high-demand articles to the floor and low-demand articles to the elevated racking, we make frequently picked articles more accessible to our coworkers, reducing unnecessary forklift use and improving our overall picking efficiency.

The vertical bar graph in the top right allows us to view pallet picks week-over-week to determine if PALLET picking demand is consistent over time or concentrated within certain weeks. The horizontal bar graph on the bottom right displays the top 10 PALLET articles by pick count, allowing us to quickly identify which articles are generating the most PALLET picks.

## Picking Heat Map and Tooltip

<img width="727" height="727" alt="image" src="https://github.com/user-attachments/assets/cb99fe43-690c-41fe-8b14-2813e4370cc9" />

The **Picking Heat Map** provides a visual representation of picking activity throughout our warehouse by aisle and bin location. Conditional formatting is used to highlight high and low activity locations, while the "IKEA" and "EURO" identifiers at the top of the visual allow us to distinguish which aisles are "IKEA-length" (long) and "Euro-length" (short) locations.

Each heat map cell also includes a custom tooltip that provides additional details about the articles assigned to each aisle and bin location. Hovering over a cell displays the articles stored in that location, allowing the user to investigate a spot without leaving the heat map.

<img width="765" height="727" alt="image" src="https://github.com/user-attachments/assets/8b247bde-2d3e-4531-ba51-442cea5e97fc" />

In the example above, we see that there are two articles located in Aisle 30, Bin 65. These two articles combine for a total of 162 picks during a set time period.

## Data Ingestion

This dashboard pulls data from three separate reports that provide information about the articles, such as picking/sales history, article name, article number, and assigned location in the warehouse. These reports are stored as Excel files in SharePoint which are set to auto export every day to their SharePoint folders, replacing the file from the previous day and allowing the Power BI to ingest the updated data every morning. Power BI connects to these SharePoint folders as its data sources, allowing the semantic model to receive the updated data.

*(Historical sales data, forecasting, and data source table names have been redacted for data privacy reasons)*

### Source Data

The three reports mentioned above are briefly described below. For confidentiality, the internal report names have been replaced with descriptive names throughout this project.

| Source | Purpose |
| --- | --- |
| **Article Dimensions** | Provides article dimensions and other physical product attributes used for pallet-size classification. |
| **Sales Space Optimization** | Provides current article and warehouse-location information used to identify where articles are stored. |
| **Picking Reports** | Historical order-picking data used to calculate article demand and picking activity by warehouse location. |

## Data Transformation

Before building the data model, I used Power Query to clean up and standardize the source data. Just as I did in the **warehouse-location-optimization** repository, I needed to ensure consistent formatting for article numbers and location IDs.

Our article numbers use eight-digit numerical identifiers. Some of my source reports omitted leading zeros for this identifier, so I used `Text.PadStart` to fix this.

```DAX
= Table.AddColumn(Table1_Table, "ARTNO_fixed", each Text.PadStart(Text.From([ARTNO]), 8, "0"))
```

Then, because of how the **Picking Reports** data is retrieved, the report can sometimes generate duplicate rows. To find these duplicate rows, I used the **Remove Duplicates** function in Power Query based on specific values in each row that can identify a unique picking record.

```DAX
= Table.Distinct(#"Changed Type", {"Order No", "Order Type", "Article No", "Pick Area", "User Picking"})
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

The **Full Serve Locations** table contains one row for each warehouse location and connects to both **Sales Space Optimization** and **Picking Reports** through the `SLID` field, allowing updated warehouse location information to be compared with the picking data to provide the aisle and bin numbers used to build the **Picking Heat Map** tab.

Finally, a **Calendar** table connects to the **Picking Reports** table using the date each article was picked, allowing picking data to be filtered and analyzed across any specific range of dates.
