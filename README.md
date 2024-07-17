
# Sales Dashboard with Copilot

### Dashboard Link : https://app.powerbi.com/groups/ca83050f-9b9a-49c1-a4a8-645047910e0d/reports/5c7bf513-bc64-44d1-b92c-4b35a24598e2/f2288446caedea92dbd9?experience=power-bi

## Problem Statement

Develop a comprehensive Sales Dashboard that offers real-time insights into key performance metrics and trends. This tool will empower stakeholders to monitor and analyze sales operations across various parameters such as budget, channel, and products. Additionally, it will provide a detailed analysis of the performance of individual salespersons and managers.

### Steps followed 

- Step 1 : Load data into Power BI Desktop, connecting through MS Excel. There were 4 different tables of which two were fact tables, together making the Galaxy Schema under data modeling.
        
        Four tables are:
        1. Fact Table (Transaction Details)
        2. Fact Table Budget (salesperson wise budget details)
        3. Product Table (Product Details)
        4. Product URL Table (Product Image Detail)
- Step 2 : Open the power query editor & in the view tab under the Data preview section, check on "column distribution", "column quality" & "column profile" options.
- Step 3 : Also since by default, the profile will be opened only for 1000 rows so you need to select "column profiling based on the entire dataset".   
- Step 4 : It was observed that within columns empty values were present, which were removed. The first columns are used as headers and data is uploaded to powerBI.
- Step 5 : Under the 'Quick Measure' Option, "Suggestions with Copilot" was used to calculate total revenue. 
        The following DAX expression was given by the copilot.

        Total Revenue = SUMX('FactTable', 'FactTable'[UnitPrice] * 'FactTable'[Quantity])

![Copilot_Revenue](https://github.com/user-attachments/assets/f5c5ddf4-5464-448a-9b29-aa11cef3f777)

- Step 6 : Under the budget table there were 3 different tables for each year. Within columns, there were null values and grand total which were removed. All the data was in a matrix table which was unpivoted and appended under a single budget facttable. Table Name is FactTable_Budget.
- Step 7 : Total Budget was calculated using copilot. 
           The following DAX expression was given by the copilot.
        
        Total Budget = SUM('FactTable_Budget'[Budget])

![Copilot_Total_Budget](https://github.com/user-attachments/assets/6ec18ab0-7d6a-409e-bccb-00ff88e857ef)

- Step 8 : A separate measure table was created to keep all the calculated measures in one place. 
- Step 9 : Another separate table was created under the power query editor, extracting the Salesperson_key, Salesperson, Supervisor, and Manager from the facttable. This table will work as a dimension table keeping all the unique records of salespersons. Later Salesperson image URLs were also added to the same table for simplicity which was used to create a salesperson tooltip. This comes under normalization. The table Name is Dim_SalesPerson. 

![Screenshot (99)](https://github.com/user-attachments/assets/b058497e-3aff-4bb5-8309-714020b362be)

- Step 10 : A separate calendar is created for date references. Weekends and Weekdays are also extracted from the same calendar.
        
        Calender = ADDCOLUMNS(
        CALENDARAUTO(),
        "Year",YEAR([Date]),
        "Month",FORMAT([Date],"MMM"),
        "MonthNum",MONTH([Date]),
        "Weekday",FORMAT([Date],"DDD"),
        "WeekNum",WEEKDAY([Date]),
        "Qtr",FORMAT([Date],"\QQ")
        )


        Weekend/Weekday = 
        IF(Calender[WeekNum] IN {1,7},"Weekend","Weekday")


- Step 11 : To get the dynamic numbers to select for top/bottom product/group/salesperson as mentioned in step-13 (I), A new parameter was created named 'Choose Rank' with minimum 1, maximum 15, increment 1, and default 1.

- Step 12 : Created a table to choose 'Top' or 'Bottom' from, named as 'Choose Rank Type', as mentioned in step-13 (I).

- Step 13 : A new parameter was created including Product Name, Product Group, and Salesperson to choose for the dynamic ranking, as mentioned in step-13 (I).

- Step 14 : Connections among the tables got established using one-to-many relationships.
Snap of Model View:
![Screenshot (100)](https://github.com/user-attachments/assets/dfed5485-db51-40ac-9ac3-8352cdf0d583)

- Step 15 : Different measures were created using the copilot same as shown above.

        A) Transaction Count

                # Transactions = COUNTROWS('FactTable')


        B) Total Quantity Sold

                Total Qty Sold = SUM('FactTable'[Quantity])


        C) Percentage of Total Quantity

                % of Qty = 
                VAR _Filtered_Value = CALCULATE(
                [Total Qty Sold],REMOVEFILTERS(Dim_SalesPerson[Salesperson])
                )

                VAR _Measure_Value = [Total Qty Sold]

                RETURN 
                DIVIDE(_Measure_Value,_Filtered_Value)


        D)Percentage of Total Revenue

                % of Revenue = 
                VAR _Filtered_Value = CALCULATE(
                [Total Revenue],
                 REMOVEFILTERS(Dim_SalesPerson)
                )

                VAR _Measure_Value = [Total Revenue]

                RETURN 
                DIVIDE(_Measure_Value,_Filtered_Value)


        E) Percentage of Revenue vs Budget

                % Revenue vs Budget = 
                DIVIDE([Total Revenue],[Total Budget])


        F) Conditional Formatting

                Conditional Formatting 1 = 
                VAR _Revenue = [Total Revenue]
                var _Budget = [Total Budget]
                VAR _Result = SWITCH(TRUE(), _Revenue < _Budget, "#A60303", "#03258C")
                
                RETURN
                _Result


        G) Ranking salesperson as per Sales

                Ranking SalesPerson = RANKX(ALL(Dim_SalesPerson),[Total Revenue],,DESC)


        H) Revenue from Budget

                Revenue from Budget = [Total Revenue]-[Total Budget]


        I) Dynamic ranking created to rank top and bottom Salesperson, ProductName, ProductGroup as per the Revenue

                Ranking = 
                VAR _Top_Product = RANKX(ALL(Dim_Product[ProductName]),[Total Revenue],,DESC)
                VAR _Bottom_Product = RANKX(ALL(Dim_Product[ProductName]),[Total Revenue],,ASC)

                VAR _Top_Salesperson = RANKX(ALL(Dim_SalesPerson[Salesperson]),[Total Revenue],,DESC)
                VAR _Bottom_Salesperson = RANKX(ALL(Dim_SalesPerson[Salesperson]),[Total Revenue],,ASC)

                VAR _Top_ProductGroup = RANKX(ALL(Dim_Product[ProductGroup]),[Total Revenue],,DESC)
                VAR _Bottom_ProductGroup = RANKX(ALL(Dim_Product[ProductGroup]),[Total Revenue],,ASC)

                VAR _CheckRank=IF(CONTAINSSTRING(SELECTEDVALUE(SelectToRank[SelectToRank Fields]),"ProductName"),
                IF(SELECTEDVALUE('Choose RankType'[Select])="Top",_Top_Product,_Bottom_Product),

                IF(CONTAINSSTRING(SELECTEDVALUE(SelectToRank[SelectToRank Fields]),"ProductGroup"),
                IF(SELECTEDVALUE('Choose RankType'[Select])="Top",_Top_ProductGroup,_Bottom_ProductGroup),

                IF(CONTAINSSTRING(SELECTEDVALUE(SelectToRank[SelectToRank Fields]),"SalesPerson"),
                IF(SELECTEDVALUE('Choose RankType'[Select])="Top",_Top_Salesperson,_Bottom_Salesperson)
                )
                )
                )

                RETURN
                IF(_CheckRank <= 'Choose Rank'[Choose Rank Value],[Total Revenue)                


        J) Conditional Formatting to Highlight the month column in Line and Stacked Column Chart as RED in which Total Revenue is less than Total Budget.

                Conditional Formatting 1 = 
                VAR _Revenue = [Total Revenue]
                var _Budget = [Total Budget]
                VAR _Result = SWITCH(TRUE(),_Revenue < _Budget,"#A60303","#03258C")

                RETURN
                _Result


        K) Sutitle to show in clustered bar Chart

                Subtitle = 
                VAR _RankType =SELECTEDVALUE('Choose RankType'[Select])
                VAR _RankNum =SELECTEDVALUE('Choose Rank'[Choose Rank])
                VAR _selectedCategory =IF(SELECTEDVALUE(SelectToRank[SelectToRank Order])=0,"Product",
                                        IF(SELECTEDVALUE(SelectToRank[SelectToRank Order])=1,"ProductGroup","Salesperson"))

                RETURN
                _RankType&"-"& _RankNum&" "&_selectedCategory&" " &"Selected"


- Step 16 : Three card visuals were added to the canvas, those representing "Total Revenue", "Total Quantity Sold", and "Total Transaction".

- Step 17 : Visuals used to represent total revenue as per different categories are mentioned below,

  (a) Total Revenue, % of Revenue, Total Quantity Sold, % of Quantity as per Manager and Salesperson (Table)

  (b) Total Revenue VS Total Budget (Line and Stacked Column Chart)
  
  (c) Total Revenue by Channel (Donut Chart)
  
  (d) Total Revenue by top/Bottom Product Name, Product Group, and Salesperson (clustered Bar Chart)
  
  (e) Total Revenue by Manager (Donut Chart)
  

- Step 18 : Three Visual filters (Slicers) were added for the clustered bar chart named "Choose Rank", "Select", "SelectToRank". Choose Rank will allow you to choose the number of products/groups/salespersons you want for the chart. Select will allow you to choose whether you wish to choose from the top or bottom. SelectToRank will allow you to choose between the product name, product group, and salesperson you want to rank.

- Step 19 : Conditional Formatting has been done on the table created using the background color and data bars to get a better understanding of the values.

- Step 20 : Customized backgrounds were created for the report view, tooltips and salespersonwise sales details using MS PowerPoint and saved in PNG format.

![Screenshot (102)](https://github.com/user-attachments/assets/278b8f94-7214-4ca9-8a48-14becc8c4ba8)

![Screenshot (101)](https://github.com/user-attachments/assets/4873b573-4d77-4a82-b9a1-0b7d501344b3)

- Step 21 : Three different tooltips were created for a more detailed view and analysis. Three tooltips are 
        
        1. Salesperson tooltip (Visible on [Total Revenue by Channel (Donut Chart)])
        2. Budget VS Revenue Tooltip (Visible on [Total Revenue VS Total Budget (Line and Stacked Column Chart)])
        3. Product Tooltip (Visible on [Total Revenue by top/Bottom Product Name, Product Group, and Salesperson (clustered Bar Chart)])

Snap of all three tooltip

![Tooltip](https://github.com/user-attachments/assets/c484573e-1d67-4300-971f-0ca9dd9ac15e)

![Tooltip2](https://github.com/user-attachments/assets/ea96ca93-88cd-48b9-906d-90711091e49c)

![Tooltip1](https://github.com/user-attachments/assets/2ed1d4d4-83f8-45bd-8d66-a83168afd258)

![Tooltip3](https://github.com/user-attachments/assets/f5a67497-5fce-495e-a546-a026907cb00c)


- Step 22 : A new page named 'Salesperson Details' was added to show the details as per different salespersons in the tabular format which include Salesperson's Image, Salesperson's Name, Total Transaction, Total Quantity Sold, total Revenue. Conditional Formatting has been done on the table created using the data bars to get a better understanding of the values.

- Step 23 : Two different Slicers were added to filter data as per the manager and supervisor on 'Salesperson Detail' Page.

- Step 24 : A Page Navigator was added to move between 'Dashboard' and 'Salesperson Detail'.

 Snap of the 'Salesperson Details' page:

 ![Salesperson Details](https://github.com/user-attachments/assets/bfd50d9c-909e-4f68-a068-bf65c59d6e63)
 
 - Step 25 : The report was then published to Power BI Service.
 
 ![Screenshot (112)](https://github.com/user-attachments/assets/3864cddb-2e3b-42ea-ad27-80ee9958e70e)

# Snapshot of Dashboard (Power BI Service)

![PowerBI01](https://github.com/user-attachments/assets/b76f63ef-288a-46bf-830b-085fd20723bc)

![PowerBI02](https://github.com/user-attachments/assets/671b494a-af86-42a1-911b-fbc9dab7da03)

 
 # Report Snapshot (Power BI DESKTOP)

![Dashboard01](https://github.com/user-attachments/assets/1fead5d4-0b26-4c74-a999-3677e5b22c19)

![Dashboard02](https://github.com/user-attachments/assets/a1086183-ae57-4975-ae3d-afdc3d60b3fc)


# Insights

Ctaegory wise Analysis:

• Top 5 Product Group contributing almost 70%.

• Bottom 5 is contributing even less than 1%.

• More than 82% of the sale comes from Retail and Distributor only. Focus on online channel is needed as it has huge potential.

• Initial months of the year have low sales and struggle to exceed the budget.

• For half of the year revenue was unable to exceed budget. Budget re-allocation can be suggested.

Overview YTD:

• Overall revenue is 17.9 M

• Total quantity sold is 6.3 M

• Total transaction count is 0.26 M

Salesperson performance:

• Top 5 salespersons contributing more than 75%.

• Bottom 5 salespersons contributing less than 15%.

• Carla Ferreira is the top salesperson followed by Julio Lima and Gustavo Gomes.

• Manager Gabriel Azevedo is contributing more with 51.44%.
