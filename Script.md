# Attribute classification SQL
### I have attached the sql script in the repo as well
```SQL
/*
Purpose: This script assigns an attribute to Hazmat battery and Hazmat product based on rules built by OPR- Dangerous Goods
Created: October 22, 2023 by Donbosco Johnson
Server: `wf-gcp-us-ae-opr-prod`
Last updated:
Procedure Name (if applicable): Hazmat_Attribute_Assignment.bq
Builds Table (if applicable): `wf-gcp-us-ae-ops-prod.csn_whs_reporting.tbl_opra_hazmat_classification`
 
Data dictionary:
https://infohub.corp.wayfair.com/pages/viewpage.action?pageId=959549857
 
Data nuances (if applicable):
(1) Hazmat product Rules- https://docs.google.com/s
(2) Hazmat battery Rules- https://docs.google.com/s
(2) Issues/ Assump- https://docs.google.com/sp
 
Version history (if applicable):
Note (if applicable):
*/
```
```SQL
CREATE OR REPLACE TEMP TABLE tmp_hazmat_products AS


--SprID and ManufacturerPartIDs belonging to a SKU would follow the same attribute mapping as that of the SKU
WITH sku_mapping_hz AS
(
  SELECT SprID
  , ManufacturerPartID
  , p.SKU
  FROM `wf-gcp-us-ae-ops-prod.analyticstech_reporting.tbl_dim_product`, UNNEST (Product) p
  WHERE p.SKU IN
  (
    SELECT prsku
    FROM `wf-gcp-us-ae-ops-prod.csn_whs_reporting.tbl_opra_hazmat`
    GROUP BY 1
  )
  GROUP BY 1,2,3
)


SELECT prsku
, s.SprID
, s.ManufacturerPartID
, prname
, prdatenew
, prstatusname
, Status_Reason
, SupplierID
, suname
, ClInternalRef
, clid
, PrBclgID
, CAST(NULL AS STRING) AS Battery_or_Battery_Included
, Haz_Goods
, UN_ID_NUMBER
, Proper_Shipping_Name
, Hazard_Class
, Packing_Group
, Packing_Instructions
, Special_Provisions_Exemptions
, Haz_Mat_Weight
, CAST(NULL AS STRING) AS Battery_Comp
, CAST(NULL AS STRING) AS Lithium_Battery
, CAST(NULL AS STRING) AS Lithium_Battery_Type
, CAST(NULL AS STRING) AS Number_Cells_Batts
, CAST(NULL AS STRING) AS Weight_Cells_Batts
, CAST(NULL AS STRING) AS Battery_Shipment
, CAST(NULL AS STRING) AS Watt_Rating
, CAST(NULL AS STRING) AS Lithium_Content
, CAST(NULL AS STRING) AS State_of_Charge
, CAST(NULL AS STRING) AS Accidnetal_Activation_Prevention
, CASE WHEN Haz_Goods = 'Yes' THEN
    CASE WHEN Hazard_Class IN ('1', '2.3', '4.2', '4.3', '6.2', '7') THEN 'Restricted for Sale'
    WHEN Hazard_Class = '2.1' THEN
      CASE WHEN clid NOT IN (881, 900, 989, 1189, 6229, 7392, 7499, 5976, 7258, 1192,6386) THEN
        CASE WHEN Haz_Mat_Weight IN ('>1L - ≤5L', '>5L') THEN 'Restricted for Sale'
        WHEN Haz_Mat_Weight IN ('>0 - ≤25ml', '>25 - ≤125ml', '>125ml - ≤500ml', '>0.5L - ≤1L') and UN_ID_NUMBER not in ('UN3090','UN3091','UN3480','UN3481','UN2800','UN3496') THEN 'Limited Quantity'
        END  
        WHEN clid IN (881, 900, 989, 1189, 6229, 7392, 7499, 5976, 7258, 1192) AND
        Haz_Mat_Weight IN ('>0 - ≤25ml', '>25 - ≤125ml', '>125ml - ≤500ml', '>0.5L - ≤1L', '>1L - ≤5L', '>5L') THEN 'Excepted REFRIGERATING MACHINES, flammable'
      END
    WHEN Hazard_Class = '2.2' THEN
      CASE WHEN clid NOT IN (881, 900, 989, 1189, 6229, 7392, 7499, 5976, 7258, 1192,6386) THEN
        CASE WHEN Haz_Mat_Weight IN ('>1L - ≤5L', '>5L') THEN 'Restricted for Sale'
        WHEN Haz_Mat_Weight IN ('>0 - ≤25ml', '>25 - ≤125ml', '>125ml - ≤500ml', '>0.5L - ≤1L') and UN_ID_NUMBER not in ('UN3090','UN3091','UN3480','UN3481','UN2800','UN3496') THEN 'Limited Quantity'
        END  
        WHEN clid IN (881, 900, 989, 1189, 6229, 7392, 7499, 5976, 7258, 1192) AND
        Haz_Mat_Weight IN ('>0 - ≤25ml', '>25 - ≤125ml', '>125ml - ≤500ml', '>0.5L - ≤1L', '>1L - ≤5L', '>5L') THEN 'Excepted REFRIGERATING MACHINES, non-flammable'
        WHEN clid IN (6386) AND
        UN_ID_NUMBER = 'UN1013' THEN 'UN1013 Soda Maker'
      END      
    WHEN Hazard_Class = '3' THEN
      CASE WHEN Packing_Group = 'I' THEN
        CASE WHEN Haz_Mat_Weight IN ('>0.5L - ≤1L', '>1L - ≤5L', '>5L') THEN 'Restricted for Sale'
        WHEN Haz_Mat_Weight IN ('>0 - ≤25ml', '>25 - ≤125ml', '>125ml - ≤500ml') and UN_ID_NUMBER not in ('UN3090','UN3091','UN3480','UN3481','UN2800','UN3496') THEN  'Limited Quantity'
        END  
      WHEN Packing_Group = 'II' THEN
        CASE WHEN Haz_Mat_Weight IN ('>1L - ≤5L', '>5L') THEN 'Restricted for Sale'
        WHEN Haz_Mat_Weight IN ('>0 - ≤25ml', '>25 - ≤125ml', '>125ml - ≤500ml', '>0.5L - ≤1L') and UN_ID_NUMBER not in ('UN3090','UN3091','UN3480','UN3481','UN2800','UN3496') THEN  'Limited Quantity'
        END
      WHEN Packing_Group = 'III' THEN
        CASE WHEN Haz_Mat_Weight IN ('>5L') THEN 'Restricted for Sale'
        WHEN Haz_Mat_Weight IN ('>0 - ≤25ml', '>25 - ≤125ml', '>125ml - ≤500ml', '>0.5L - ≤1L', '>1L - ≤5L') and UN_ID_NUMBER not in ('UN3090','UN3091','UN3480','UN3481','UN2800','UN3496') THEN  'Limited Quantity'
        END
      END
    WHEN Hazard_Class = '4.1' THEN
      CASE WHEN Packing_Group = 'I' THEN 'Restricted for Sale'  
      WHEN Packing_Group = 'II' THEN
        CASE WHEN Haz_Mat_Weight IN ('>1kg - ≤5kg', '>5kg') THEN 'Restricted for Sale'
        WHEN Haz_Mat_Weight IN ('>0 - ≤100g', '>100 - ≤500g', '>500g - ≤1kg') and UN_ID_NUMBER not in ('UN3090','UN3091','UN3480','UN3481','UN2800','UN3496') THEN  'Limited Quantity'
        END
      WHEN Packing_Group = 'III' THEN
        CASE WHEN Haz_Mat_Weight IN ('>5kg') THEN 'Restricted for Sale'
        WHEN Haz_Mat_Weight IN ('>0 - ≤100g', '>100 - ≤500g', '>500g - ≤1kg', '>1kg - ≤5kg') and UN_ID_NUMBER not in ('UN3090','UN3091','UN3480','UN3481','UN2800','UN3496') THEN  'Limited Quantity'
        END
      END
    WHEN Hazard_Class = '5.1' THEN
      CASE WHEN Packing_Group = 'I' THEN 'Restricted for Sale'
      WHEN Packing_Group = 'II' THEN
        CASE WHEN Haz_Mat_Weight IN ('>1L - ≤5L', '>5L', '>1kg - ≤5kg', '>5kg') THEN 'Restricted for Sale'
        WHEN Haz_Mat_Weight IN ('>0 - ≤25ml', '>25 - ≤125ml', '>125ml - ≤500ml', '>0.5L - ≤1L', '>0 - ≤100g', '>100 - ≤500g', '>500g - ≤1kg') and UN_ID_NUMBER not in ('UN3090','UN3091','UN3480','UN3481','UN2800','UN3496') THEN  'Limited Quantity'
        END
      WHEN Packing_Group = 'III' THEN
        CASE WHEN Haz_Mat_Weight IN ('>5L', '>5kg') THEN 'Restricted for Sale'
        WHEN Haz_Mat_Weight IN ('>0 - ≤25ml', '>25 - ≤125ml', '>125ml - ≤500ml', '>0.5L - ≤1L', '>1L - ≤5L', '>0 - ≤100g', '>100 - ≤500g', '>500g - ≤1kg', '>1kg - ≤5kg') and UN_ID_NUMBER not in ('UN3090','UN3091','UN3480','UN3481','UN2800','UN3496') THEN  'Limited Quantity'
        END
      END
    WHEN Hazard_Class = '5.2' THEN
      CASE WHEN Packing_Group = 'I' THEN 'Restricted for Sale'
      WHEN Packing_Group = 'II' THEN
        CASE WHEN Haz_Mat_Weight IN ('>0 - ≤25ml', '>0 - ≤100g') and UN_ID_NUMBER not in ('UN3090','UN3091','UN3480','UN3481','UN2800','UN3496') THEN  'Limited Quantity'
        WHEN Haz_Mat_Weight IN ('>25 - ≤125ml', '>125ml - ≤500ml', '>0.5L - ≤1L', '>1L - ≤5L', '>5L', '>100 - ≤500g', '>500g - ≤1kg', '>1kg - ≤5kg', '>5kg') THEN 'Restricted for Sale'
        END
      WHEN Packing_Group = 'III' THEN
        CASE WHEN Haz_Mat_Weight IN ('>0 - ≤25ml', '>25 - ≤125ml', '>0 - ≤100g', '>100 - ≤500g') and UN_ID_NUMBER not in ('UN3090','UN3091','UN3480','UN3481','UN2800','UN3496') THEN  'Limited Quantity'
        WHEN Haz_Mat_Weight IN ('>125ml - ≤500ml', '>0.5L - ≤1L', '>1L - ≤5L',  '>5L', '>500g - ≤1kg', '>1kg - ≤5kg', '>5kg') THEN 'Restricted for Sale'
        END
      END
    WHEN Hazard_Class = '6.1' THEN
      CASE WHEN Packing_Group = 'I' THEN 'Restricted for Sale'
      WHEN Packing_Group = 'II' THEN
        CASE WHEN Haz_Mat_Weight IN ('>0 - ≤25ml', '>25 - ≤125ml', '>0 - ≤100g') and UN_ID_NUMBER not in ('UN3090','UN3091','UN3480','UN3481','UN2800','UN3496') THEN  'Limited Quantity'
        WHEN Haz_Mat_Weight IN ('>125ml - ≤500ml', '>0.5L - ≤1L', '>1L - ≤5L', '>5L', '>100 - ≤500g', '>500g - ≤1kg', '>1kg - ≤5kg', '>5kg') THEN 'Restricted for Sale'
        END
      WHEN Packing_Group = 'III' THEN
        CASE WHEN Haz_Mat_Weight IN ('>5L', '>5kg') and UN_ID_NUMBER not in ('UN3090','UN3091','UN3480','UN3481','UN2800','UN3496') THEN  'Limited Quantity'
        WHEN Haz_Mat_Weight IN ('>0 - ≤25ml', '>25 - ≤125ml', '>125ml - ≤500ml', '>0.5L - ≤1L', '>1L - ≤5L', '>0 - ≤100g', '>100 - ≤500g', '>500g - ≤1kg', '>1kg - ≤5kg')  THEN 'Restricted for Sale'
        END
      END
    WHEN Hazard_Class IN ('8', '9') THEN
      CASE WHEN Packing_Group = 'I' THEN 'Restricted for Sale'
      WHEN Packing_Group = 'II' THEN
        CASE WHEN Haz_Mat_Weight IN ('>1L - ≤5L', '>5L', '>1kg - ≤5kg', '>5kg') THEN  'Restricted for Sale'
        WHEN Haz_Mat_Weight IN ('>0 - ≤25ml', '>25 - ≤125ml', '>125ml - ≤500ml', '>0.5L - ≤1L',  '>0 - ≤100g', '>100 - ≤500g', '>500g - ≤1kg') and UN_ID_NUMBER not in ('UN3090','UN3091','UN3480','UN3481','UN2800','UN3496') THEN 'Limited Quantity'
        END
      WHEN Packing_Group = 'III' THEN
        CASE WHEN Haz_Mat_Weight IN ('>5L', '>5kg') THEN  'Restricted for Sale'
        WHEN Haz_Mat_Weight IN ('>0 - ≤25ml', '>25 - ≤125ml', '>125ml - ≤500ml', '>0.5L - ≤1L', '>1L - ≤5L',  '>0 - ≤100g', '>100 - ≤500g', '>500g - ≤1kg', '>1kg - ≤5kg') and UN_ID_NUMBER not in ('UN3090','UN3091','UN3480','UN3481','UN2800','UN3496') THEN 'Limited Quantity'
        END
      END
    END --hazard class
  END --hazard goods
AS Attribute
, CASE WHEN Haz_Goods = 'Yes' AND Hazard_Class IS NOT NULL THEN 'Global' ELSE NULL END AS Region --applicable to all rules where haz good exist and class
, CASE WHEN Haz_Goods = 'Yes' THEN 'Hazmat product' ELSE NULL END AS Product_Type --adding this rule to distinguish between hazmat products and batteries
FROM  `wf-gcp-us-ae-ops-prod.csn_whs_reporting.tbl_opra_hazmat`  hz
LEFT JOIN sku_mapping_hz s ON hz.prsku = s.SKU
WHERE  1 =1
AND  Haz_Goods = 'Yes'
GROUP BY 1,2,3,4,5,6,7,8,9,10,11,12,13,14,15,16,17,18,19,20,21;




CREATE OR REPLACE TEMP TABLE tmp_hazmat_batteries AS
--SprID and ManufacturerPartIDs belonging to a SKU would follow the same attribute mapping as that of the SKU
WITH sku_mapping AS
(
  SELECT SprID
  , ManufacturerPartID
  , p.SKU
  FROM `wf-gcp-us-ae-ops-prod.analyticstech_reporting.tbl_dim_product`, UNNEST (Product) p
  WHERE p.SKU IN
  (
    SELECT prsku
    FROM `wf-gcp-us-ae-ops-prod.csn_whs_reporting.tbl_opra_hazmat_batteries`
    GROUP BY 1
  )
  GROUP BY 1,2,3
)


SELECT DISTINCT
prsku
, s.SprID
, s.ManufacturerPartID
, prname
, prdatenew
, prstatusname
, Status_Reason
, SupplierID
, suname
, ClInternalRef
, clid
, PrBclgID
, Battery_or_Battery_Included
, CAST(NULL AS STRING) AS Haz_Goods
, CAST(NULL AS STRING) AS UN_ID_NUMBER
, CAST(NULL AS STRING) AS Proper_Shipping_Name
, CAST(NULL AS STRING) AS Hazard_Class
, CAST(NULL AS STRING) AS Packing_Group
, CAST(NULL AS STRING) AS Packing_Instructions
, CAST(NULL AS STRING) AS Special_Provisions_Exemptions
, CAST(NULL AS STRING) AS Haz_Mat_Weight
, Battery_Comp
, Lithium_Battery
, Lithium_Battery_Type
, Number_Cells_Batts
, Weight_Cells_Batts
, Battery_Shipment
, Watt_Rating
, Lithium_Content
, State_of_Charge
, Accidnetal_Activation_Prevention
, CASE
  WHEN Lithium_Battery = 'Yes' THEN --not dependent on battery or battery included question


    CASE
    WHEN Watt_Rating IN ('Batteries of between 100w/h and 300w/h','Batteries of between 100w/h and 300 w/h', 'Batteries exceeding 300w/h', 'Cells of between 20w/h and 60w/h', 'Cells exceeding 60 w/h') THEN 'Restricted for Sale'
    WHEN Lithium_Content IN ('Batteries of between 2 and 25 grams', 'Batteries exceeding 25 grams', 'Cells of between 1 and 5 grams', 'Cells exceeding 5 grams') THEN 'Restricted for Sale'


    WHEN Lithium_Battery_Type = 'Battery' THEN -----BATTERY  


      CASE
      --WHEN Watt_Rating = 'Batteries of 100w/h or less' THEN 'Excepted lithium batteries UN3481'----------------------------------------------------> FLOW III   (BWHR) COMMENT OUT
      --WHEN Watt_Rating IN ('Batteries of between 100w/h and 300w/h', 'Batteries exceeding 300w/h') THEN 'Restricted for Sale' ----------------------------------------------------> FLOW III  (BWHR)


      -- WHEN Lithium_Content = 'Batteries of 2 grams or less' THEN 'Excepted lithium batteries UN3091' ----------------------------------------------------> FLOW III   (LC)  COMMENT OUT
      --WHEN Lithium_Content IN ('Batteries of between 2 and 25 grams', 'Batteries exceeding 25 grams') THEN 'Restricted for Sale'----------------------------------------------------> FLOW III   (LC)


      WHEN Number_Cells_Batts = 'More than 2 batteries or 4 cells' THEN -------------------------------------> FLOW III  (NOC/B= M)
        CASE WHEN Battery_Shipment IN ( 'Contained in equipment' , 'Contained in product' )THEN
          CASE WHEN Battery_Comp = 'Lithium Ion / Lithium Polymer' THEN
            CASE WHEN Watt_Rating = 'Batteries of 100w/h or less' THEN 'Excepted lithium batteries UN3481'   -------------> CHECK FOR DROPDOWN NAMES
            END
          WHEN Battery_Comp = 'Lithium Metal' THEN
             CASE WHEN Lithium_Content = 'Batteries of 2 grams or less' THEN 'Excepted lithium batteries UN3091'
             END
          END
        WHEN Battery_Shipment = 'Packed with product' THEN 'Restricted for sale'
        END                                                    ----------------------------------------------------> FLOW III  (NOC/B= M)
      --WHEN Watt_Rating = 'Batteries of 100w/h or less' THEN 'Excepted lithium batteries UN3481'
      WHEN Number_Cells_Batts IN ( 'Less than 2 batteries or 4 cells', '2 batteries or less / 4 cells or less') AND Battery_Shipment NOT IN ('Stand alone') THEN -------------------------------------> FLOW III       (NOC/B= L)
        CASE WHEN Battery_Shipment = 'Packed with product' THEN
          CASE WHEN Battery_Comp = 'Lithium Ion / Lithium Polymer' THEN
            CASE WHEN Watt_Rating = 'Batteries of 100w/h or less' THEN 'Excepted lithium batteries UN3481'   -------------> CHECK FOR DROPDOWN NAMES
            END
          WHEN Battery_Comp = 'Lithium Metal' THEN
             CASE WHEN Lithium_Content = 'Batteries of 2 grams or less' THEN 'Excepted lithium batteries UN3091'
             END
          END
        WHEN Battery_Shipment IN ( 'Contained in equipment' , 'Contained in product' )THEN
          CASE WHEN Watt_Rating = 'Batteries of 100w/h or less' THEN 'Excepted lithium batteries UN3481'
          WHEN Lithium_Content = 'Batteries of 2 grams or less' THEN 'Excepted lithium batteries UN3091'
          END
        END                                                   ----------------------------------------------------> FLOW III   (NOC/B= L)
      WHEN Battery_Shipment = 'Stand alone' THEN ----------------------------------------------------> FLOW III   (BS)
        CASE WHEN Battery_Comp = 'Lithium Ion / Lithium Polymer' THEN
          CASE WHEN Watt_Rating = 'Batteries of 100w/h or less' THEN 'Excepted lithium batteries UN3480'   -------------> CHECK FOR DROPDOWN NAMES
          END
        WHEN Battery_Comp = 'Lithium Metal' THEN
           CASE WHEN Lithium_Content = 'Batteries of 2 grams or less' THEN 'Excepted lithium batteries UN3090'
           END
        END                                      ----------------------------------------------------> FLOW III   (BS)
      END


    WHEN Lithium_Battery_Type = 'Cell' THEN  --CELL


      CASE
                      -- WHEN Watt_Rating = 'Cells of 20w/h or less' THEN 'Excepted lithium batteries UN3481'----------------------------------------------------> FLOW III   (BWHR) COMMENT OUT
                      --WHEN Watt_Rating IN ('Cells of between 20w/h and 60w/h', 'Cells exceeding 60 w/h') THEN 'Restricted for Sale' ----------------------------------------------------> FLOW III  (BWHR)
                      -- WHEN Lithium_Content = 'Cells of 1 gram or less' THEN 'Excepted lithium batteries UN3091' ----------------------------------------------------> FLOW III   (LC)  COMMENT OUT
                      --WHEN Lithium_Content IN ('Cells of between 1 and 5 grams', 'Cells exceeding 5 grams') THEN 'Restricted for Sale'----------------------------------------------------> FLOW III   (LC)


      WHEN Number_Cells_Batts = 'More than 2 batteries or 4 cells' THEN -------------------------------------> FLOW III  (NOC/B= M)
        CASE WHEN Battery_Shipment IN ( 'Contained in equipment' , 'Contained in product' ) THEN
          CASE WHEN Battery_Comp = 'Lithium Ion / Lithium Polymer' THEN
            CASE WHEN Watt_Rating = 'Cells of 20w/h or less' THEN 'Excepted lithium batteries UN3481'   -------------> CHECK FOR DROPDOWN NAMES
            END
          WHEN Battery_Comp = 'Lithium Metal' THEN
             CASE WHEN Lithium_Content = 'Cells of 1 gram or less' THEN 'Excepted lithium batteries UN3091'
             END
          END
        WHEN Battery_Shipment = 'Packed with product' THEN 'Restricted for sale'
        END                                                    ----------------------------------------------------> FLOW III  (NOC/B= M)


      WHEN Number_Cells_Batts IN ( 'Less than 2 batteries or 4 cells', '2 batteries or less / 4 cells or less') AND Battery_Shipment NOT IN ('Stand alone') THEN -------------------------------------> FLOW III       (NOC/B= L)
        CASE WHEN Battery_Shipment = 'Packed with product' THEN
          CASE WHEN Battery_Comp = 'Lithium Ion / Lithium Polymer' THEN
            CASE WHEN Watt_Rating = 'Cells of 20w/h or less' THEN 'Excepted lithium batteries UN3481'   -------------> CHECK FOR DROPDOWN NAMES
            END
          WHEN Battery_Comp = 'Lithium Metal' THEN
             CASE WHEN Lithium_Content = 'Cells of 1 gram or less' THEN 'Excepted lithium batteries UN3091'
             END
          END
        WHEN Battery_Shipment IN ( 'Contained in equipment' , 'Contained in product' ) THEN
          CASE WHEN Watt_Rating ='Cells of 20w/h or less' THEN 'Excepted lithium batteries UN3481'
          WHEN Lithium_Content = 'Cells of 1 gram or less' THEN 'Excepted lithium batteries UN3091'
          END


        END                                                    ----------------------------------------------------> FLOW III   (NOC/B= L)


      WHEN Battery_Shipment = 'Stand alone' THEN ----------------------------------------------------> FLOW III   (BS)
        CASE WHEN Battery_Comp = 'Lithium Ion / Lithium Polymer' THEN
         CASE WHEN Watt_Rating = 'Cells of 20w/h or less' THEN 'Excepted lithium batteries UN3480'   -------------> CHECK FOR DROPDOWN NAMES
         END
        WHEN Battery_Comp = 'Lithium Metal' THEN
           CASE WHEN Lithium_Content = 'Cells of 1 gram or less' THEN 'Excepted lithium batteries UN3090'
           END
        END                                      ----------------------------------------------------> FLOW III   (BS)
      END


    WHEN Lithium_Battery_Type = 'Button Cell' THEN  ----BUTTON CELL


      CASE
                      -- WHEN Watt_Rating = 'Cells of 20w/h or less' THEN 'Excepted lithium batteries UN3481'----------------------------------------------------> FLOW III   (BWHR) COMMENT OUT
                      --WHEN Watt_Rating IN ('Cells of between 20w/h and 60w/h', 'Cells exceeding 60 w/h') THEN 'Restricted for Sale' ----------------------------------------------------> FLOW III  (BWHR)
                      -- WHEN Lithium_Content = 'Cells of 1 gram or less' THEN 'Excepted lithium batteries UN3091' ----------------------------------------------------> FLOW III   (LC)  COMMENT OUT
                      --WHEN Lithium_Content IN ('Cells of between 1 and 5 grams', 'Cells exceeding 5 grams') THEN 'Restricted for Sale'----------------------------------------------------> FLOW III   (LC)


      WHEN Number_Cells_Batts IN ( 'Less than 2 batteries or 4 cells', '2 batteries or less / 4 cells or less') AND Battery_Shipment NOT IN ('Stand alone') THEN -------------------------------------> NEW CHANGE
        CASE WHEN Battery_Shipment = 'Packed with product' THEN
          CASE WHEN Battery_Comp = 'Lithium Ion / Lithium Polymer' THEN 'Excepted lithium batteries UN3481' -------------> CHECK FOR DROPDOWN NAMES
          WHEN Battery_Comp = 'Lithium Metal' THEN 'Excepted lithium batteries UN3091'
          END
        END  
      WHEN Number_Cells_Batts = 'More than 2 batteries or 4 cells' THEN
        CASE WHEN Battery_Shipment = 'Packed with product' THEN 'Restricted for sale'
        END


      WHEN Battery_Shipment = 'Stand alone' THEN ------------------------------>NEW CHANGE
        CASE WHEN Battery_Comp = 'Lithium Ion / Lithium Polymer' THEN
          CASE WHEN Watt_Rating = 'Cells of 20w/h or less' THEN  'Excepted lithium batteries UN3480' -------------> CHECK FOR DROPDOWN NAMES
          END
        WHEN Battery_Comp = 'Lithium Metal' THEN
          CASE WHEN  Lithium_Content = 'Cells of 1 gram or less' THEN  'Excepted lithium batteries UN3090'
          END
        END                                      ------------------------------>NEW CHANGE    
      END
    END -- schema tag #2
    WHEN Battery_or_Battery_Included = 'Yes' THEN
      CASE WHEN Battery_Comp = 'Lead Acid (spillable)' THEN 'Restricted for Sale'
      WHEN Battery_Comp IN ('Lead Acid (Non spillable)','Lead Acid (Non-Spillable)') THEN 'Non-spillable batteries'
      WHEN Battery_Comp = 'Nickel Metal Hydride' THEN 'Not Restricted' END     ----------------------> NEW CHANGE I
  END --schema tag #1 - battery included ; lithium battery
AS Attribute
, CASE WHEN Battery_or_Battery_Included = 'Yes' OR Lithium_Battery = 'Yes' THEN 'Global' ELSE NULL END AS Region
, CASE WHEN Battery_or_Battery_Included = 'Yes' OR Lithium_Battery = 'Yes' THEN 'Hazmat battery' ELSE NULL END AS Product_Type


FROM `wf-gcp-us-ae-ops-prod.csn_whs_reporting.tbl_opra_hazmat_batteries` hb
LEFT JOIN sku_mapping s ON hb.prsku = s.SKU
WHERE 1 =1
AND (Battery_or_Battery_Included = 'Yes' OR Lithium_Battery = 'Yes');






CREATE OR REPLACE TABLE `wf-gcp-us-ae-ops-prod.csn_whs_reporting.tbl_opra_hazmat_classification` AS
SELECT hz.*
--Issues: New addition to identify issues with misclassified products
, CASE WHEN Attribute IS NULL THEN
    CASE WHEN Hazard_Class NOT IN ('1', '2.1', '2.2', '2.3', '3', '4.1', '4.2', '4.3', '5.1', '5.2', '6.1', '6.2', '7', '8', '9') OR Hazard_Class IS NULL THEN 'Issue with Hazard Class'
    -- WHEN Hazard_Class IN ('1', '2.3', '4.2', '4.3', '6.2', '7') THEN 'Should have had been classified'
    WHEN Hazard_Class IN ('2.1', '2.2') THEN 'Possible issue with class ID or weight'
    WHEN Hazard_Class IN ('3', '4.1', '5.1', '5.2', '6.1', '8', '9') THEN
      CASE WHEN Packing_Group NOT IN ('I','II','III') OR Packing_Group IS NULL THEN 'Issue with Packing Group'
      WHEN Packing_Group IN ('I','II','III') THEN 'Possible issue with weight'
      END
    ELSE 'Unknown Issue'
    END
  WHEN Attribute IS NOT NULL THEN 'No Issue'
  END AS Issue
FROM tmp_hazmat_products hz


UNION ALL


SELECT hb.*
, CASE WHEN Attribute IS NULL THEN
    CASE
    WHEN Battery_or_Battery_Included = 'Yes'  AND (Lithium_Battery NOT IN ('Yes') OR Lithium_Battery IS NULL) THEN
        CASE
        WHEN (Battery_Comp  IS NULL or Battery_Comp = "[N/A]") AND Lithium_Battery  IS NULL AND Lithium_Battery_Type  IS NULL AND Number_Cells_Batts  IS NULL AND Battery_Shipment  IS NULL AND Watt_Rating  IS NULL AND Lithium_Content  IS NULL THEN 'BATT YES BUT REST ALL NULL'
        WHEN Lithium_Battery IS NOT NULL AND Battery_Comp NOT IN ('Lead Acid (spillable)',  'Lead Acid (Non spillable)', 'Nickel Metal Hydride','Lead Acid (Non-Spillable)') OR Battery_Comp IS NULL THEN 'Issue with battery composition'
        WHEN Battery_Comp IN ('Lithium Ion / Lithium Polymer','Lithium Metal') THEN 'Li battery = NO but Batt Comp = Li ion/metal'   END  --- not listed in our baterry composition (will give out typos )
        WHEN Battery_Comp NOT IN ('Lithium Ion / Lithium Polymer','Lithium Metal') THEN 'Issue with battery composition'
       
    WHEN Lithium_Battery = 'Yes' THEN
      CASE
      WHEN Lithium_Battery_Type NOT IN ('Battery', 'Cell', 'Button Cell') OR Lithium_Battery_Type IS NULL THEN 'Issue with Lithium battery type'
      WHEN Lithium_Battery_Type IN ('Battery', 'Cell', 'Button Cell') THEN 'Possible issue with Number of cells or Watt Rating or Lithium Content'
      WHEN Battery_Shipment NOT IN ('Stand alone','Packed with product','Contained in equipment' , 'Contained in product') AND Battery_Shipment IS NULL THEN 'Issue with Battery Shipment'  END --break this down
    ELSE 'Unknown Issue'
    END
  WHEN Attribute IS NOT NULL THEN 'No Issue'
  END AS Issue
FROM tmp_hazmat_batteries hb;
```
