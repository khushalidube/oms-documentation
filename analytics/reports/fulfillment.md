---
description: Discover Fulfillment reports provided by HotWax Commerce
---

# Fulfillment

## Rejected Order Items Report

It is common for fulfillment locations to reject online orders due to various factors such as inventory discrepancies or fulfillment issues. However, minimizing these rejections holds importance for retailers striving to uphold customer satisfaction and streamline operations. HotWax Commerce recognizes this need and offers a report on rejected orders from multi-fulfillment locations.

This report serves as a crucial tool for retailers in their efforts to reduce rejection rates. By providing insights into rejection occurrences—including timing, locations, SKUs, and order IDs—it empowers users to identify patterns and root causes behind rejections. Armed with this information, retailers can take measures to optimize inventory management, streamline fulfillment processes, and enhance staff training.

HotWax Commerce's rejection report enables users to make decisions aimed at improving overall operational efficiency. By understanding which items are rejected more frequently and in which locations, retailers can implement strategies to minimize rejections and ensure accurate, timely order fulfillment. Ultimately, this proactive approach not only enhances customer satisfaction but also strengthens the retailer's competitive edge in the market.

### Glossary

| Item                  | Item Details                                                 | HC Entity                                                                                             |
| --------------------- | ------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------- |
| **Shopify Order ID**  | Identifies the order within the Shopify platform             | OrderHeader.ORDER\_NAME                                                                               |
| **Location**          | Indicates the location of the order fulfillment              | OrderFacilityChange.FROM\_FACILITY\_ID                                                                |
| **HC Order ID**       | HotWax Commerce Order ID for internal tracking               | OrderHeader.ORDER\_ID                                                                                 |
| **Style**             | Specifies the style or type of the product                   | OrderItem.ITEM\_DESCRIPTION                                                                           |
| **Demand**            | Reflects the demand or quantity ordered                      | OrderItem.QUANTITY                                                                                    |
| **SKU**               | Stock Keeping Unit for inventory tracking                    | GoodIdentification.ID\_VALUE where GoodIdentification.GOOD\_IDENTIFICATION\_TYPE\_ID is set to ‘UPCA’ |
| **Product Name**      | Name of the product ordered                                  | Product.PRODUCT\_NAME                                                                                 |
| **ATP**               | Indicates the quantity of the product available for shipment | ProductFacility.LAST\_INVENTORY\_COUNT                                                                |
| **Safety Stock**      | Quantity of stock kept in reserve to meet unexpected demand  | ProductFacility.MINIMUM\_STOCK                                                                        |
| **Rejected Datetime** | A timestamp indicating when the rejection occurred           | OrderFacilityChange.CHANGE\_DATETIME                                                                  |
| **Reason**            | Specifies the reason for rejection                           | OrderFacilityChange.CHANGE\_REASON\_ENUM\_ID                                                          |
| **Comments**          | Additional comments or notes regarding the rejection         | OrderFacilityChange.COMMENTS                                                                          |

<details>

<summary>SQL Query to Generate Rejected Order Items Report</summary>

```sql
SELECT `Shopify Order ID` AS `Shopify Order ID`,
       `Location` AS `Location`,
       `HC Order Id` AS `HC Order Id`,
       `Style` AS `Style`,
       `SKU` AS `SKU`,
       `Product Name` AS `Product Name`,
       `ATP` AS `ATP`,
       `Safety Stock ` AS `Safety Stock `,
       `Rejected_Datetime` AS `Rejected_Datetime`,
       `Reason` AS `Reason`,
       `Comments` AS `Comments`
FROM
  (SELECT OFC.FROM_FACILITY_ID AS 'Location',
          OH.ORDER_NAME AS 'Shopify Order ID',
          OH.ORDER_ID AS 'HC Order Id',
          OI.ITEM_DESCRIPTION AS 'Style',
     (SELECT giupca.ID_VALUE
      FROM good_identification giupca
      WHERE oi.PRODUCT_ID = giupca.PRODUCT_ID
        AND giupca.GOOD_IDENTIFICATION_TYPE_ID = 'UPCA'
        AND (giupca.THRU_DATE > NOW() OR giupca.THRU_DATE IS NULL)
      LIMIT 1) AS SKU,
          P.PRODUCT_NAME AS 'Product Name',
          PF.LAST_INVENTORY_COUNT AS 'ATP',
          PF.MINIMUM_STOCK AS 'Safety Stock ',
          OFC.CHANGE_DATETIME AS Rejected_Datetime,
          OFC.CHANGE_REASON_ENUM_ID AS 'Reason',
          OFC.COMMENTS AS 'Comments',
                       F.FACILITY_ID AS FACILITY_ID,
                       OS.STATUS_DATETIME AS COMPLETED_DATETIME
   FROM ORDER_HEADER OH
   INNER JOIN ORDER_ITEM OI ON OH.ORDER_ID = OI.ORDER_ID
   INNER JOIN ORDER_FACILITY_CHANGE OFC ON OFC.ORDER_ID = OI.ORDER_ID
   AND OFC.ORDER_ITEM_SEQ_ID = OI.ORDER_ITEM_SEQ_ID
   AND OFC.CHANGE_REASON_ENUM_ID NOT IN ('BROKERED', 'UNFILLABLE', 'RELEASED')
   INNER JOIN PRODUCT P ON P.PRODUCT_ID = OI.PRODUCT_ID
   INNER JOIN PRODUCT_FACILITY PF ON PF.PRODUCT_ID = OI.PRODUCT_ID
   AND PF.FACILITY_ID = OFC.FROM_FACILITY_ID
   INNER JOIN FACILITY F ON PF.FACILITY_ID = F.FACILITY_ID
   AND F.FACILITY_TYPE_ID IN ('WAREHOUSE', 'RETAIL_STORE')
   LEFT JOIN ORDER_STATUS OS ON OS.ORDER_ID = OI.ORDER_ID
   AND OS.ORDER_ITEM_SEQ_ID = OI.ORDER_ITEM_SEQ_ID
   AND OS.STATUS_ID = "ITEM_COMPLETED"
   WHERE OH.PRODUCT_STORE_ID = 'STORE'
   ORDER BY OFC.CHANGE_DATETIME DESC) AS virtual_table
WHERE `Rejected_Datetime` >= STR_TO_DATE('2024-05-06 00:00:00.000000', '%Y-%m-%d %H:%i:%s.%f')
  AND `Rejected_Datetime` < STR_TO_DATE('2024-05-13 00:00:00.000000', '%Y-%m-%d %H:%i:%s.%f')
LIMIT 50000;
```

</details>

### Query Logic

**Data Selection:** The SQL query begins by selecting specific data from various tables relevant to generating a Rejected Order Items Report. These tables include information about orders, order items, order statuses, and product details.

**Joining Relevant Tables:** To compile a comprehensive dataset, the SQL query joins relevant tables together based on common fields such as order IDs and product IDs. By doing so, it gathers interconnected information about rejected order items, including order details, item descriptions, rejection reasons, and external IDs.

**Selecting Necessary Information:** From the joined tables, the query selects specific columns relevant to the report's objective. These columns typically include order IDs, rejection dates, SKUs (product identifiers), item descriptions, rejection reasons, and external IDs. This selection provides essential details required for analyzing and understanding rejected order items.

**Grouping Data:** Subsequently, the query groups the selected data based on certain criteria such as order ID, rejection date, SKU, item description, rejection reason, and external ID. Grouping the data helps in summarizing and organizing it for further analysis, facilitating insights into patterns and trends related to rejected order items.

**Counting Distinct Orders:** Within each group, the query counts the number of distinct order IDs. This count provides insights into the volume of unique orders that had items rejected, helping understand the overall impact of rejection on order fulfillment.

**Calculating Date:** Additionally, the query includes a calculation to determine the date of the rejected order items. It may derive the first day of the week for each rejection date, facilitating grouping of rejections by week and enabling a broader perspective on rejection trends over time.

## Store Rejections with Reasons

Rejection reason reports provide valuable insights into rejection rates and reasons across different facilities. These reports outline the number of rejections at each facility along with the underlying reasons behind them. This allows retailers to pinpoint facilities with higher rejection rates and take targeted actions to enhance their performance. For example, if a store frequently experiences rejections due to items being out of stock, conducting cycle counts can help reconcile discrepancies between the system's available inventory and the physical stock in the store. Moreover, by identifying facilities where damaged items are more prevalent, retailers can streamline operations to mitigate such occurrences. Ultimately, the aim of these reports is to pinpoint the location and frequency of rejections and implement measures to minimize them, thereby optimizing overall store performance.

### Glossary

| Item                        | Item Details                                                             | HC Entity                                    |
| --------------------------- | ------------------------------------------------------------------------ | -------------------------------------------- |
| **Location**                | The physical location or facility from which an order item was rejected. | OrderFacilityChange.FROM\_FACILITY\_ID       |
| **Reason**                  | The reason for the rejection of an order item.                           | OrderFacilityChange.CHANGE\_REASON\_ENUM\_ID |
| **COUNT(Shopify Order ID)** | The count of Shopify order IDs associated with each rejection reason.    | OrderHeader.ORDER\_NAME                      |

<details>

<summary>SQL Query to Generate Store Rejections with Reasons</summary>

```sql
SELECT `Location` AS `Location`,
       `Reason` AS `Reason`,
       count(`Shopify Order ID`) AS `COUNT(Shopify Order ID)`
FROM
  (SELECT OFC.FROM_FACILITY_ID AS 'Location',
          OH.ORDER_NAME AS 'Shopify Order ID',
          OH.ORDER_ID AS 'HC Order Id',
          OI.ITEM_DESCRIPTION AS 'Style',

     (select giupca.ID_VALUE
      from good_identification giupca
      where oi.PRODUCT_ID = giupca.PRODUCT_ID
        and giupca.GOOD_IDENTIFICATION_TYPE_ID = 'UPCA'
        and (giupca.THRU_DATE > NOW()
             OR giupca.THRU_DATE IS NULL)
      limit 1) as SKU,
          P.PRODUCT_NAME AS 'Product Name',
          PF.LAST_INVENTORY_COUNT AS 'ATP',
          PF.MINIMUM_STOCK AS 'Safety Stock ',
          OFC.CHANGE_DATETIME AS Rejected_Datetime,
          OFC.CHANGE_REASON_ENUM_ID AS 'Reason',
          OFC.COMMENTS AS 'Comments',
                       F.FACILITY_ID AS FACILITY_ID,
                       OS.STATUS_DATETIME AS COMPLETED_DATETIME
   FROM ORDER_HEADER OH
   INNER JOIN ORDER_ITEM OI ON OH.ORDER_ID = OI.ORDER_ID
   INNER JOIN ORDER_FACILITY_CHANGE OFC ON OFC.ORDER_ID=OI.ORDER_ID
   AND OFC.ORDER_ITEM_SEQ_ID=OI.ORDER_ITEM_SEQ_ID
   AND OFC.CHANGE_REASON_ENUM_ID NOT IN ('BROKERED',
                                         'UNFILLABLE',
                                         'RELEASED')
   INNER JOIN PRODUCT P ON P.PRODUCT_ID=OI.PRODUCT_ID
   INNER JOIN PRODUCT_FACILITY PF ON PF.PRODUCT_ID=OI.PRODUCT_ID
   AND PF.FACILITY_ID=OFC.FROM_FACILITY_ID
   INNER JOIN FACILITY F ON PF.FACILITY_ID=F.FACILITY_ID
   and F.FACILITY_TYPE_ID IN('WAREHOUSE',
                             'RETAIL_STORE')
   LEFT JOIN ORDER_STATUS OS ON OS.ORDER_ID = OI.ORDER_ID
   AND OS.ORDER_ITEM_SEQ_ID=OI.ORDER_ITEM_SEQ_ID
   AND OS.STATUS_ID = "ITEM_COMPLETED"
   WHERE OH.PRODUCT_STORE_ID = 'STORE'
   ORDER BY OFC.CHANGE_DATETIME DESC) AS virtual_table
WHERE `Rejected_Datetime` >= STR_TO_DATE('2024-05-06 00:00:00.000000', '%Y-%m-%d %H:%i:%s.%f')
  AND `Rejected_Datetime` < STR_TO_DATE('2024-05-13 00:00:00.000000', '%Y-%m-%d %H:%i:%s.%f')
GROUP BY `Location`,
         `Reason`
ORDER BY `COUNT(Shopify Order ID)` DESC
LIMIT 5000;
```

</details>

### Query Logic

**Data Selection:** The SQL query is designed to gather information for generating a report on Store Rejections with Reasons. It selects specific data from various tables, including details about orders, order items, order statuses, and product information. These tables store interconnected data related to store orders and inventory.

**Joining Relevant Tables:** To compile a comprehensive dataset, the SQL query joins relevant tables together based on common fields such as order IDs, product IDs, and facility IDs. By joining these tables, it gathers detailed information about rejected order items, including order details, item descriptions, rejection reasons, and facility locations.

**Selecting Necessary Information:** From the joined tables, the query selects specific columns relevant to the report's objective. These columns typically include the location (store), rejection reasons, and the count of Shopify order IDs associated with each rejection reason. This selection provides essential details required for analyzing and understanding store rejections.

**Grouping Data:** Subsequently, the query groups the selected data based on certain criteria such as location (store) and rejection reason. Grouping the data allows for summarization and organization, facilitating insights into common rejection reasons across different store locations.

**Counting Rejected Orders:** Within each group, the query counts the number of distinct Shopify order IDs. This count provides insights into the volume of rejected orders associated with each rejection reason and location, aiding in understanding the frequency and impact of rejections.

**Sorting and Limiting Results:** Finally, the query sorts the results based on the count of Shopify order IDs in descending order. This arrangement helps prioritize the most significant rejection reasons. Additionally, it limits the output to a maximum of 5000 records, ensuring manageable data for analysis.

## Shipment Tracking Report

Merchants use third-party services to ship orders to customers' addresses. Orders are picked, packed, and handed to third-party logistics (3PL) companies like FedEx and UPS. All shipments dispatched have their unique tracking ID. This tracking ID is provided by the 3PLs and helps customers keep track of their orders. Because multiple shipments are sent out daily, merchants require a centralized view of order shipment details. This Shopify report helps merchants quickly identify an order's tracking ID, shipping address, and dispatch location. The Shipment Tracking Report shows the daily shipment details for each order. Using this report, merchants can quickly resolve questions related to order tracking by finding the tracking ID and the shipping address for the order.

### Glossary

| Item                  | Item Details                                                                                                                                                            | HC Entity                                                                                                                         |
| --------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| **Shopify Id**        | The ID of Order in Shopify.                                                                                                                                             | OrderHeader.ORDER\_NAME                                                                                                           |
| **Tracking Number**   | The tracking number of the shipped order.                                                                                                                               | ShipmentPackageRouteSegment.TRACKING\_CODE                                                                                        |
| **Shipping Method**   | The shipping method used for the order shipment.                                                                                                                        | OrderItemShipGroup.SHIPMENT\_METHOD\_TYPE\_ID                                                                                     |
| **Shipping Location** | The code of the fulfillment location.                                                                                                                                   | OrderItemShipGroup.FACILITY\_ID                                                                                                   |
| **Ship To Address**   | The shipping address of the customer. Concatenation (Appended string of) PostalAddress.ADDRESS1, PostalAddress.STATE\_PROVINCE\_GEO\_ID and PostalAddress.POSTAL\_CODE) | Concatenation (Appended string of) PostalAddress.ADDRESS1, PostalAddress.STATE\_PROVINCE\_GEO\_ID and PostalAddress.POSTAL\_CODE) |
| **UNITS SHIPPED**     | The number of units shipped in order.                                                                                                                                   | Total count of OrderItem.ORDER\_ITEM\_SEQ\_ID                                                                                     |

<details>

<summary>SQL Query to Generate Shipment Tracking Report</summary>

```sql
SELECT `Shopify Id` AS `Shopify Id`,
       `Tracking Number` AS `Tracking Number`,
       `Shipping Method` AS `Shipping Method`,
       `Shipping Location` AS `Shipping Location`,
       `Ship To Address` AS `Ship To Address`,
       `UNITS SHIPPED` AS `UNITS SHIPPED`
FROM
  (SELECT oh.ORDER_NAME AS 'Shopify Id',
          trck.TRACKING_CODE AS 'Tracking Number',
          oisg.SHIPMENT_METHOD_TYPE_ID AS 'Shipping Method',
          oisg.FACILITY_ID AS 'Shipping Location',
          concat(pa.ADDRESS1, ' ', pa.STATE_PROVINCE_GEO_ID, ' ', pa.POSTAL_CODE) AS 'Ship To Address',
          count(oi.ORDER_ITEM_SEQ_ID) AS 'UNITS SHIPPED',
          oh.PRODUCT_STORE_ID,
          oh.ORDER_TYPE_ID
   FROM order_item oi
   JOIN order_header oh ON oi.order_id = oh.order_id
   LEFT JOIN order_shipment osh ON oi.ORDER_ID = osh.ORDER_ID AND oi.ORDER_ITEM_SEQ_ID = osh.ORDER_ITEM_SEQ_ID
   LEFT JOIN shipment_status ss ON osh.SHIPMENT_ID = ss.SHIPMENT_ID
   LEFT JOIN shipment s ON osh.SHIPMENT_ID = s.SHIPMENT_ID
   JOIN order_item_ship_group oisg ON oi.ORDER_ID = oisg.ORDER_ID AND oi.SHIP_GROUP_SEQ_ID = oisg.SHIP_GROUP_SEQ_ID
   LEFT JOIN (SELECT oi.order_id,
                     oi.order_item_seq_id,
                     sprs.tracking_code AS 'TRACKING_CODE'
              FROM order_item oi
              INNER JOIN order_shipment os ON oi.ORDER_ID = os.ORDER_ID AND oi.ORDER_ITEM_SEQ_ID = os.ORDER_ITEM_SEQ_ID
              INNER JOIN shipment_package_route_seg sprs ON os.SHIPMENT_ID = sprs.SHIPMENT_ID AND sprs.TRACKING_CODE IS NOT NULL) trck ON oi.ORDER_ID = trck.ORDER_ID AND oi.ORDER_ITEM_SEQ_ID = trck.ORDER_ITEM_SEQ_ID
   LEFT JOIN order_contact_mech ocm ON oi.ORDER_ID = ocm.ORDER_ID
   INNER JOIN postal_address pa ON ocm.CONTACT_MECH_ID = pa.CONTACT_MECH_ID
   WHERE ss.STATUS_ID = 'SHIPMENT_SHIPPED'
     AND oi.STATUS_ID = 'ITEM_COMPLETED'
     AND oh.PRODUCT_STORE_ID = 'STORE'
     AND trck.TRACKING_CODE IS NOT NULL
   GROUP BY trck.TRACKING_CODE) AS virtual_table
WHERE `Shipped Date` >= STR_TO_DATE('2024-05-06 00:00:00.000000', '%Y-%m-%d %H:%i:%s.%f')
  AND `Shipped Date` < STR_TO_DATE('2024-05-13 00:00:00.000000', '%Y-%m-%d %H:%i:%s.%f')
  AND `ORDER_TYPE_ID` = 'SALES_ORDER'
GROUP BY `Shopify Id`,
         `Tracking Number`,
         `Shipping Method`,
         `Shipping Location`,
         `Ship To Address`,
         `UNITS SHIPPED`
LIMIT 10;
```

</details>

### Query Logic

**Data Selection:** The SQL query is structured to extract specific data for generating a Shipment Tracking Report. It selects information from various tables, and containing details about orders, order items, shipments, shipment statuses, and shipping addresses.

**Defining Shipment Criteria:** The primary objective of the report is to track shipments. Therefore, the query defines criteria to filter the data, including only shipments that have been marked as shipped (`STATUS_ID = 'SHIPMENT_SHIPPED'`) and associated order items that are completed (`STATUS_ID = 'ITEM_COMPLETED'`).

**Joining Relevant Tables:** To gather comprehensive shipment data, the SQL query joins several tables together based on common fields such as order IDs, order item sequence IDs, and shipment IDs. By doing so, it collects interconnected information about shipments, order items, shipment statuses, and shipping addresses.

**Selecting Necessary Information:** From the joined tables, the query selects specific columns relevant to the report's objective. These columns typically include the Shopify order ID, tracking number, shipping method, shipping location, ship-to address, and the number of units shipped. This selection provides essential details required for tracking and analyzing shipments.

**Grouping Data:** Subsequently, the query groups the selected data based on certain criteria such as the tracking number. Grouping the data allows for summarization and organization, facilitating insights into the status and progress of each shipment.

**Counting Units Shipped:** Within each group, the query counts the number of units shipped. This count provides insights into the volume of items included in each shipment, aiding in understanding shipment sizes and fulfillment metrics.

**Sorting and Limiting Results:** Finally, the query sorts the results based on the Shopify order ID and limits the output to a maximum of 10 records. This arrangement helps prioritize and present the most relevant shipment tracking information while ensuring a manageable dataset for review.

## Shopify Fulfillment Status Report

The Shopify Fulfillment Status Report offers retailers valuable insights into their order processing. It highlights the number of orders fulfilled and those still pending, aiding in the assessment of operational efficiency. Retailers can leverage reports from Hotwax to access historical data and current statistics, enabling them to track performance over time and promptly address any issues. It serves as a progress report for order fulfillment, ensuring smooth operations.

### Glossary

| Item                               | Item Details                                                          | HC Entity                                                                                             |
| ---------------------------------- | --------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------- |
| **Hotwax Order ID**                | Unique identifier for the order in Hotwax system                      | OrderFulfillmentHistory.ORDER\_ID                                                                     |
| **Shopify Order Name**             | Name or identifier associated with the order in Shopify system        | OrderHeader.ORDER\_NAME                                                                               |
| **SKU**                            | Stock Keeping Unit, a unique code assigned to each product            | Product.INTERNAL\_NAME                                                                                |
| **Facility Name**                  | Name of the facility where inventory is stored                        | Facility.FACILITY\_NAME                                                                               |
| **Item Status**                    | Status of the item (e.g., available, out of stock, backordered)       | StatusItem.DESCRIPTION                                                                                |
| **Hotwax Shipment Status**         | Status of the shipment in Hotwax                                      | StatusItem.DESCRIPTION                                                                                |
| **Hotwax Fulfillment Time**        | Time taken to fulfill the order in Hotwax                             | ShipmentStatus.STATUS\_DATETIME                                                                       |
| **Created Time**                   | Time when the order was created                                       | OrderFulfillmentHistory.CREATED\_DATE                                                                 |
| **Shopify Fulfillment ID**         | Unique identifier for the fulfillment process in Shopify system       | OrderFulfillmentHistory.EXTERNAL\_FULFILLMENT\_ID                                                     |
| **Shopify Fulfillment Status**     | Status of the fulfillment process in Shopify system                   | Set to ‘pending’ if OrderFulfillmentHistory.EXTERNAL\_FULFILLMENT\_ID is set to '_NA_' else ‘success’ |
| **Shopify Fulfillment Entry Time** | Time when the fulfillment process was initiated in the Shopify system | OrderFulfillmentHistory.LAST\_UPDATED\_STAMP                                                          |

<details>

<summary>SQL Query to Generate Shopify Fulfillment Status Since Last Day Pie Chart</summary>

```sql
SELECT shopify_fulfillment_status AS shopify_fulfillment_status,
       COUNT(shopify_fulfillment_status) AS `COUNT(shopify_fulfillment_status)`
FROM
  (SELECT ofh.ORDER_ID AS hotwax_order_id ,
          oh.ORDER_NAME AS shopify_order_name ,
          p.INTERNAL_NAME AS sku ,
          f.FACILITY_NAME ,
          f.FACILITY_ID AS FACILITY_ID ,
          si.DESCRIPTION AS item_status ,
          si2.DESCRIPTION AS hotwax_shipment_status ,
          ss.STATUS_DATE AS hotwax_fulfillment_time ,
          ofh.CREATED_DATE AS created_time ,
          CASE
              WHEN ofh.EXTERNAL_FULFILLMENT_ID = '_NA_' THEN NULL
              ELSE ofh.EXTERNAL_FULFILLMENT_ID
          END AS shopify_fulfillment_id ,
          CASE
              WHEN ofh.EXTERNAL_FULFILLMENT_ID = '_NA_' THEN 'pending'
              ELSE 'success'
          END AS shopify_fulfillment_status ,
          CASE
              WHEN ofh.EXTERNAL_FULFILLMENT_ID = '_NA_' THEN NULL
              ELSE ofh.LAST_UPDATED_STAMP
          END AS shopify_fulfillment_entry_time
   FROM order_fulfillment_history ofh
   JOIN order_header oh ON ofh.ORDER_ID = oh.ORDER_ID
   AND oh.ORDER_TYPE_ID = "SALES_ORDER"
   JOIN order_item oi ON ofh.ORDER_ID = oi.ORDER_ID
   AND ofh.ORDER_ITEM_SEQ_ID = oi.ORDER_ITEM_SEQ_ID
   JOIN status_item si ON oi.STATUS_ID = si.STATUS_ID
   JOIN product p ON oi.PRODUCT_ID = p.PRODUCT_ID
   JOIN shipment s ON ofh.SHIPMENT_ID = s.SHIPMENT_ID
   JOIN status_item si2 ON s.STATUS_ID = si2.STATUS_ID
   JOIN facility f ON s.ORIGIN_FACILITY_ID = f.FACILITY_ID
   AND s.STATUS_ID = ss.STATUS_ID
   JOIN shipment_status ss ON ofh.SHIPMENT_ID = ss.SHIPMENT_ID
   WHERE ofh.SHIPMENT_ID IS NOT NULL
     AND oi.EXTERNAL_ID IS NOT NULL) AS virtual_table
WHERE ((shopify_fulfillment_status = 'pending'
        or cast(created_time as date) = cast((NOW() - Interval 1 day) as date)))
GROUP BY shopify_fulfillment_status
ORDER BY `COUNT(shopify_fulfillment_status)` DESC
LIMIT 100;
```

</details>

### Query Logic

**Data Selection:** The SQL query is crafted to gather data for generating a Shopify Fulfillment Status Since Last Day Pie Chart. It selects specific information from various tables, including details about orders, order items, order fulfillment history, shipments, statuses, products, and facilities.

**Defining Criteria:** The primary objective of the report is to analyze the fulfillment status of Shopify orders within the last day. To achieve this, the query defines criteria to filter the data, including only orders of type "SALES\_ORDER" and fulfillment history associated with shipments. Additionally, it focuses on orders where the Shopify fulfillment status is either 'pending' or has been updated within the last day.

**Joining Relevant Tables:** To compile a comprehensive dataset, the SQL query joins several tables together based on common fields such as order IDs, order item sequence IDs, shipment IDs, and status IDs. By doing so, it gathers interconnected information about order fulfillment, order items, shipments, statuses, products, and facilities.

**Selecting Necessary Information:** From the joined tables, the query selects specific columns relevant to the report's objective. These columns typically include the Shopify fulfillment status and the count of orders corresponding to each fulfillment status. This selection provides essential details required for visualizing the distribution of fulfillment statuses in the pie chart.

**Grouping Data:** Subsequently, the query groups the selected data based on the Shopify fulfillment status. Grouping the data allows for summarization and organization, facilitating insights into the distribution of pending and successfully fulfilled orders.

**Sorting and Limiting Results:** Finally, the query sorts the results based on the count of orders for each fulfillment status in descending order. This arrangement helps prioritize and present the most relevant fulfillment status information. Additionally, it limits the output to a maximum of 100 records, ensuring a manageable dataset for visualization in the pie chart.

## Daily Store Shipment Performance Report

Merchants measure their stores’ omnichannel order fulfillment rates and want a clear, accurate view of their daily shipping performance. Therefore, merchants seek a detailed storewide report that displays every store’s daily shipping performance. The Daily Store Shipment Performance Report shows the number of daily shipped orders for each store, giving an overview of the store’s daily performance. This performance report also yields valuable insight into the most active and least active stores within a set timeframe. With this data, retailers can work with underperforming stores to understand the reason for low activity, identify the cause of underperformance, and resolve the issue. Similarly, the retailer wants to see product-level shipping performance. The report also includes product-level performance. This information helps retailers understand a newly launched product's performance in a given area or region. Retailers can plan their inventory accordingly after knowing which products perform best at a given store.

### Glossary

| Item        | Item Details                                   | HC Entity                                                                                                           |
| ----------- | ---------------------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| **Store**   | The fulfillment store location of the retailer | OrderFacilityChange.FACILITY\_ID                                                                                    |
| **UPC/SKU** | The unique product code                        | GoodIdentification.ID\_VALUE where GoodIdentification.GOOD\_IDENTIFICATION\_TYPE\_ID is set to 'SHOPIFY\_PROD\_SKU' |
| **Style**   | The category of the product/item               | Product.PRODUCT\_NAME                                                                                               |
| **Color**   | The color of the product                       | ProductFacility.DESCRIPTION when ProductFeature.PRODUCT\_FEATURE\_TYPE\_ID is set to 'COLOR'                        |
| **Size**    | The size of the product                        | ProductFacility.DESCRIPTION when ProductFeature.PRODUCT\_FEATURE\_TYPE\_ID is set to ‘SIZE'                         |
| **Units**   | The units shipped of the product               | Total sum of OrderItem.QUANTITY                                                                                     |

<details>

<summary>SQL Query to Generate Daily Store Shipment Performance Report</summary>

```sql
SELECT `STORE` AS `STORE`,
       `SKU` AS `SKU`,
       `STYLE` AS `STYLE`,
       `COLOR` AS `COLOR`,
       `UNITS` AS `UNITS`,
       `SIZE` AS `SIZE`
FROM
  (SELECT ofc.facility_id 'STORE',
                          ofc.facility_id 'FACILITY_ID',
                                          vp.PRODUCT_NAME 'STYLE',

     (select giupca.ID_VALUE
      from good_identification giupca
      where oi.PRODUCT_ID = giupca.PRODUCT_ID
        and giupca.GOOD_IDENTIFICATION_TYPE_ID = 'SHOPIFY_PROD_SKU') as SKU,
                                                          pf.color 'COLOR',
                                                                   pf.size 'SIZE',
                                                                           sum(oi.quantity) 'UNITS',
                                                                                            oh.PRODUCT_STORE_ID
   FROM order_item oi
   join order_header oh on oi.ORDER_ID = oh.ORDER_ID
   JOIN order_facility_change ofc ON oi.order_id = ofc.order_id
   AND oi.order_item_seq_id = ofc.order_item_seq_id
   JOIN order_status os ON oi.order_id = os.ORDER_ID
   AND oi.ORDER_ITEM_SEQ_ID = os.ORDER_ITEM_SEQ_ID
   INNER JOIN
     (SELECT ORDER_ID,
             ORDER_ITEM_SEQ_ID,
             max(CHANGE_DATETIME) 'MaxDate'
      FROM order_facility_change ofc2
      GROUP BY ORDER_ID,
               ORDER_ITEM_SEQ_ID) tm ON ofc.ORDER_ID = tm.ORDER_ID
   AND ofc.ORDER_ITEM_SEQ_ID = tm.ORDER_ITEM_SEQ_ID
   AND ofc.CHANGE_DATETIME = tm.MaxDate
   JOIN facility fac ON ofc.FACILITY_ID = fac.FACILITY_ID
   LEFT JOIN
     (SELECT PRODUCT_ID,
             min(CASE
                     WHEN pf.PRODUCT_FEATURE_TYPE_ID = 'COLOR' THEN pf.DESCRIPTION
                 END) AS 'color' ,
             min(CASE
                     WHEN pf.PRODUCT_FEATURE_TYPE_ID = 'SIZE' THEN pf.DESCRIPTION
                 END) AS 'size'
      FROM product_feature_appl pfa
      JOIN product_feature pf ON pf.PRODUCT_FEATURE_ID = pfa.PRODUCT_FEATURE_ID
      AND pf.PRODUCT_FEATURE_TYPE_ID IN ('SIZE',
                                         'COLOR')
      GROUP BY PRODUCT_ID) pf ON pf.PRODUCT_ID = oi.PRODUCT_ID
   JOIN product p ON oi.PRODUCT_ID = p.PRODUCT_ID
   JOIN product_assoc pa ON p.PRODUCT_ID = pa.PRODUCT_ID_TO
   JOIN product vp ON pa.PRODUCT_ID = vp.PRODUCT_ID
   WHERE CAST(os.STATUS_DATETIME AS date) = (current_date() - INTERVAL 1 DAY)
     AND os.STATUS_ID = 'ITEM_COMPLETED'
     AND ofc.change_reason_enum_id = 'BROKERED'
     AND fac.FACILITY_TYPE_ID = 'RETAIL_STORE'
   GROUP BY ofc.FACILITY_ID,
            pf.color,
            pf.size,
            p.product_id,
            oh.PRODUCT_STORE_ID) AS virtual_table
LIMIT 1000;
```

</details>

### Query Logic

**Data Selection:** The SQL query is intended to generate a Daily Store Shipment Performance Report. It begins by selecting specific data from various tables, including order items, order headers, order facility changes, order statuses, facilities, product features, products, and product associations.

**Defining Criteria:** The primary objective of the report is to analyze the shipment performance of retail stores on a daily basis. To achieve this, the query sets criteria to filter the data. It focuses on orders completed on the previous day (current\_date() - INTERVAL 1 DAY), marked with a status ID of 'ITEM\_COMPLETED', and involving a change reason enum ID of 'BROKERED'. Additionally, it specifically targets retail stores by filtering facility types.

**Joining Relevant Tables:** To compile a comprehensive dataset, the SQL query joins multiple tables together based on common fields such as order IDs, order item sequence IDs, facility IDs, product IDs, and status date. By doing so, it gathers interconnected information about order items, order headers, order facility changes, order statuses, facilities, product features, products, and product associations.

**Selecting Necessary Information:** From the joined tables, the query selects specific columns relevant to the report's objective. These columns typically include store IDs, SKUs (product identifiers), styles, colors, sizes, and the number of units shipped. This selection provides essential details required for assessing store shipment performance.

**Grouping Data:** Subsequently, the query groups the selected data based on store IDs, colors, sizes, and product IDs. Grouping the data allows for summarization and organization, facilitating insights into the distribution of shipped units across different products and attributes within each store.

**Sorting and Limiting Results:** Finally, the query limits the output to a maximum of 1000 records, ensuring a manageable dataset for analysis. This step ensures that the report focuses on the most relevant information while maintaining a reasonable size for practical use.

## Store Daily Orders Fulfillment Report

The Store Daily Orders Fulfillment Report empowers retailers to monitor the efficiency and effectiveness of their store operations. By tracking the daily number of orders fulfilled per store, retailers can identify trends and patterns, facilitating individual store performance assessments. This insight aids in recognizing top-performing stores and pinpointing areas for improvement in underperforming ones. With Store Daily Orders Fulfillment Reports, retailers can optimize store operations, enhance customer satisfaction, and ultimately drive revenue growth.

### Glossary

| Item         | Item Details                           | HC Entity                                                                                            |
| ------------ | -------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| **Store**    | Store Name                             | OrderFacilityChange.FACILITY\_ID                                                                     |
| **Count**    | Count of orders fulfilled in a store   | Total count of OrderFacilityChange.ORDER\_ITEM\_SEQ\_ID where facilities are of type ‘RETAIL\_STORE’ |
| **Quantity** | Total items/units fulfilled by a store | Total sum of OrderItem.QUANTITY grouped by facilities that are ‘STORE’                               |

<details>

<summary>SQL Query to Generate Store Daily Order Fulfillment Report</summary>

```sql
SELECT `STORE` AS `STORE`,
       sum(`sum(oi.quantity)`) AS `SUM(sum(oi.quantity))`
FROM
  (Select ofc.facility_id 'STORE',
                          ofc.facility_id 'FACILITY_ID',
                                          count(ofc.ORDER_ITEM_SEQ_ID),
                                          sum(oi.quantity)
   from order_facility_change ofc
   join order_status os on ofc.ORDER_ID = os.ORDER_ID
   and ofc.ORDER_ITEM_SEQ_ID = os.ORDER_ITEM_SEQ_ID
   inner join
     (select ORDER_ID,
             ORDER_ITEM_SEQ_ID,
             max(CHANGE_DATETIME) 'MaxDate'
      from order_facility_change ofc2
      group by ORDER_ID,
               ORDER_ITEM_SEQ_ID) tm on ofc.ORDER_ID = tm.ORDER_ID
   and ofc.ORDER_ITEM_SEQ_ID = tm.ORDER_ITEM_SEQ_ID
   and ofc.CHANGE_DATETIME = tm.MaxDate
   join facility fac on ofc.FACILITY_ID = fac.FACILITY_ID
   join order_header oh on ofc.ORDER_ID = oh.ORDER_ID
   join order_item oi on ofc.ORDER_ID = oi.ORDER_ID
   and ofc.ORDER_ITEM_SEQ_ID = oi.ORDER_ITEM_SEQ_ID
   where cast(os.STATUS_DATETIME as date) =date(now() - INTERVAL 1 DAY)
     and os.STATUS_ID='ITEM_COMPLETED'
     and ofc.change_reason_enum_id ='BROKERED'
     and fac.FACILITY_TYPE_ID='RETAIL_STORE'
     and oh.PRODUCT_STORE_ID='STORE'
   group by ofc.FACILITY_ID) AS virtual_table
GROUP BY `STORE`
ORDER BY `SUM(sum(oi.quantity))` DESC
LIMIT 100;
```

</details>

### Query Logic

**Data Selection:** The SQL query is designed to generate a Store Daily Order Fulfillment Report. It begins by selecting specific data from various tables, including order facility changes, order statuses, facilities, order headers, and order items.

**Defining Criteria:** The primary objective of the report is to analyze the daily order fulfillment performance of retail stores. To achieve this, the query sets criteria to filter the data. It focuses on orders completed on the previous day, marked with a status ID of 'ITEM\_COMPLETED', and involving a change reason enum ID of 'BROKERED'. Additionally, it specifically targets retail stores by filtering facility types and a particular product store ID.

**Joining Relevant Tables:** To compile a comprehensive dataset, the SQL query joins multiple tables together based on common fields such as order IDs, order item sequence IDs, facility IDs, and status dates. By doing so, it gathers interconnected information about order facility changes, order statuses, facilities, order headers, and order items.

**Grouping and Summarizing Data:** The query groups the selected data based on store IDs (facility IDs) and calculates the sum of quantities of order items fulfilled for each store. Grouping and summarizing the data in this manner allows for a clear overview of the total quantity of items fulfilled by each store.

**Sorting and Limiting Results:** Finally, the query sorts the results based on the sum of quantities in descending order and limits the output to a maximum of 100 records. This step ensures that the report focuses on the stores with the highest order fulfillment quantities, providing insights into their performance while maintaining a manageable dataset for analysis.
