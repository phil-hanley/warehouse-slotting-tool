# warehouse-slotting-tool
Power BI dashboard for analyzing picking activity and sales history in the warehouse to support optimized inventory planning decisions

## Business Problem
When we are planning our warehouse floor, we must balance high and low demand articles in an efficient way. This means high demand articles should be as accessible as possible, while low demand articles should be given low priority locations. Given that our warehouse has thousands of articles and only a fraction of floor and shelf slots to fit these, we must be careful about the way we plan our warehouse articles. 

Our historical sales and order picking data are difficult to analyze in their raw form. When we are making inventory planning decisions, we typically find ourselves digging through multiple raw reports and planning without a complete view of warehouse activity. There is no simple way for us to compare the performance of every article or to see what articles whose current location may not fit their level of demand.

A poorly planned warehouse can negatively affect our order picking productivity and create frustration for coworkers, who may have to consistently take longer to pick the same article because of its location. These locations could either be too far away from our main picking area or could be stored in elevated racking that requires a forklift to retrieve. Over time, inefficient planning creates unnecessary travel time and equipment use during the picking process.

## Solution
To provide a complete, extensive view of warehouse sales activity, I developed a Power BI dashboard that combines historical picking data with article and warehouse location data. This tool provides three different pages for analyzing article demand and identifying articles that would be potential candidates for relocation.

## Article Picking Analysis

<img width="1316" height="737" alt="image" src="https://github.com/user-attachments/assets/041df670-3324-4814-b42f-e874723cafa6" />

The Article Picking Analysis page provides the picking activity of every warehouse article. User can filter articles by location type (floor or shelf), aisle, product area and division, and pallet size, as well as a range of dates to identify both high and low demand articles throughout the warehouse.

(Dates have been erased from the date 

## PALLET article performance

<img width="1672" height="941" alt="pallet picking analysis" src="https://github.com/user-attachments/assets/01ad767c-3ba4-4c6e-b6d9-c3aafce0c296" />

