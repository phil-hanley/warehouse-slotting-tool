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

(Dates have been erased from the date slicer to protect historical sales data)

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
