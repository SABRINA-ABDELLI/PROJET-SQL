# PROJET-SQL
-- Affichez par ordre décroissant d'ancienneté les employés masculins dont le salaire net (salaire + commission) est supérieur ou égal à 8000. Le tableau résultant doit inclure les colonnes suivantes : Numéro d'employé, Prénom et Nom, Âge et Ancienneté.
select * from EMPLOYEES

 select EMPLOYEE_int, FIRST_NAME, LAST_NAME, DATEDIFF(YEAR, BIRTH_DATE,
 GETDATE()) AS AGE, DATEDIFF(YEAR, HIRE_DATE, GETDATE()) AS SENIORITY
 from EMPLOYEES
 where (SALARY + Coalesce(COMMISSION,0))>= 8000 and TITLE like 'Mr.'
 order by DATEDIFF(YEAR, HIRE_DATE, GETDATE()) desc;

 --Affichez les produits répondant aux critères suivants : (C1) la quantité est conditionnée en bouteille(s), (C2) le troisième caractère du nom du produit est « t » ou « T », (C3) fourni par les fournisseurs 1, 2 ou 3, (C4) le prix unitaire est compris entre 70 et 200 ; et (C5) les unités commandées sont spécifiées (non nulles). Le tableau obtenu doit inclure les colonnes suivantes : numéro de produit, nom du produit, numéro de fournisseur, unités commandées et prix unitaire.
  select * from PRODUCTS
 SELECT
 PRODUCT_REF as 'Product Number',
 PRODUCT_NAME as 'Product Name',
 SUPPLIER_int as 'Supplier number',
 UNITS_ON_ORDER as 'units ordered',
 UNIT_PRICE as 'unit price'
 FROM PRODUCTS
 WHERE QUANTITY LIKE '%bottle%'
 AND (PRODUCT_NAME LIKE '__T%' OR PRODUCT_NAME LIKE '__t%')
 AND (SUPPLIER_int in (1,2,3))
 AND (UNIT_PRICE>= 70 and UNIT_PRICE<= 200)
 AND (UNITS_ON_ORDER is not NULL);

 SELECT PRODUCT_REF,PRODUCT_NAME, SUPPLIER_int,
 UNITS_ON_ORDER, UNIT_PRICE
 FROM PRODUCTS
 WHERE CHARINDEX('bottle', QUANTITY)>0
 AND SUBSTRING(PRODUCT_NAME, 3, 1) IN ('t', 'T')
 AND SUPPLIER_int IN (1, 2, 3)
 AND UNIT_PRICE BETWEEN 70.00 AND 200.00
 AND UNITS_ON_ORDER IS NOT NULL;

 --Affichez les clients résidant dans la même région que le fournisseur 1, c'est-à-dire partageant le même pays, la même ville et les trois derniers chiffres du code postal. La requête doit utiliser une seule sous-requête. La table résultante doit inclure toutes les colonnes de la table des clients.
 SELECT C.*
 FROM CUSTOMERS as C
 where exists ( select s.*
 from SUPPLIERS s
 where SUPPLIER_int=1
 and C.COUNTRY= s.COUNTRY
 and C.CITY= s.CITY
 and SUBSTRING(C.POSTAL_CODE,
 LEN(C.POSTAL_CODE)-2, 3) like SUBSTRING(s.POSTAL_CODE, LEN(s.POSTAL_CODE)-2,
 3));

 --Pour chaque numéro de commande compris entre 10998 et 11003, procédez comme suit :  
-- Affichez le nouveau taux de remise, qui doit être de 0 % si le montant total de la commande avant remise (prix unitaire * quantité) est compris entre 0 et 2000, 5 % s'il est compris entre 2001 et 10000, 10 % s'il est compris entre 10001 et 40000, 15 % s'il est compris entre 40001 et 80000, et 20 % dans les autres cas.
-- Affichez le message « Appliquer l'ancien taux de remise » si le numéro de commande est compris entre 10000 et 10999, et « Appliquer le nouveau taux de remise » dans les autres cas. Le tableau résultant doit afficher les colonnes : numéro de commande, nouveau taux de remise et note d'application du taux de remise.
  Select
 ORDER_int,
 (CASE
 WHEN UNIT_PRICE * QUANTITY between 0 and 2000 THEN 0
 WHEN UNIT_PRICE * QUANTITY between 2001 and 10000 THEN 5
 WHEN UNIT_PRICE * QUANTITY between 10001 and 40000 THEN 10
 WHEN UNIT_PRICE * QUANTITY between 40001 and 80000 THEN 15
ELSE 20
 END) as 'new discount rate',
 (CASE
 WHEN ORDER_int between 10000 and 10999 THEN 'apply old discount
 rate'
 ELSE 'apply new discount rate'
 END) as 'discount rate application note'
 from ORDER_DETAILS
 where ORDER_int between 10998 and 11003;

 SELECT * 
 FROM ORDER_DETAILS;

 --Affichez les fournisseurs de boissons. Le tableau obtenu doit contenir les colonnes suivantes : numéro du fournisseur, entreprise, adresse et numéro de téléphone.
 SELECT
SUPPLIERS.SUPPLIER_int,
SUPPLIERS.COMPANY,
SUPPLIERS.ADDRESS,
SUPPLIERS.PHONE
FROM
SUPPLIERS
INNER JOIN PRODUCTS ON SUPPLIERS.SUPPLIER_int = PRODUCTS.SUPPLIER_int
INNER JOIN CATEGORIES ON PRODUCTS.CATEGORY_CODE = CATEGORIES.CATEGORY_CODE
WHERE
CATEGORIES.CATEGORY_NAME = 'Beverages';
select S.SUPPLIER_int, S.COMPANY, s.ADDRESS, s.PHONE as 'phone number'
from SUPPLIERS S
join Products p on p.SUPPLIER_int = S.SUPPLIER_int
join CATEGORIES c on c.CATEGORY_CODE = P.CATEGORY_CODE
where c.CATEGORY_NAME like 'Beverages';

--Affichez les clients de Berlin ayant commandé au maximum 1 (0 ou 1) dessert. Le tableau obtenu doit afficher la colonne : code client
select C.CUSTOMER_CODE
from CUSTOMERS C
join ORDERS O on O.CUSTOMER_CODE=C.CUSTOMER_CODE
join ORDER_DETAILS OD on OD.ORDER_NUMBER=O.ORDER_NUMBER
join Products p on p.PRODUCT_REF = OD.PRODUCT_REF
join CATEGORIES cat on cat.CATEGORY_CODE = p.PRODUCT_REF
where cat.CATEGORY_NAME like 'Desserts' and c.city like 'Berlin'
group by C.CUSTOMER_CODE
having count(distinct p.PRODUCT_REF)<2 ;

--Affichez les clients résidant en France et le montant total de leurs commandes passées chaque lundi d'avril 1998 (en tenant compte des clients n'ayant pas encore passé de commande). Le tableau obtenu doit afficher les colonnes suivantes : numéro de client, nom de l'entreprise, numéro de téléphone, montant total et pays.
SELECT
CUSTOMERS.CUSTOMER_CODE,
CUSTOMERS.COMPANY,
CUSTOMERS.PHONE,
ISNULL(SUM(ORDER_DETAILS.UNIT_PRICE * ORDER_DETAILS.QUANTITY), 0) AS
TOTAL_AMOUNT,
CUSTOMERS.COUNTRY
FROM
CUSTOMERS
LEFT JOIN ORDERS ON CUSTOMERS.CUSTOMER_CODE = ORDERS.CUSTOMER_CODE
LEFT JOIN ORDER_DETAILS ON ORDERS.ORDER_int = ORDER_DETAILS.ORDER_int
WHERE
CUSTOMERS.COUNTRY = 'France'
AND ORDER_DATE BETWEEN '1998-04-01' AND '1998-04-30'
AND DATENAME(WEEKDAY, ORDER_DATE) = 'Monday'
GROUP BY
CUSTOMERS.CUSTOMER_CODE, CUSTOMERS.COMPANY, CUSTOMERS.PHONE, CUSTOMERS.COUNTRY;

--Affichez les clients ayant commandé tous les produits. Le tableau obtenu doit contenir les colonnes suivantes : code client, nom de l'entreprise et numéro de téléphone.
select C.CUSTOMER_CODE, C.COMPANY as COPANY_NAME, C.PHONE as PHONE_NUMBER -- ,
count(distinct OD.PRODUCT_REF) as Produit_acheter
from CUSTOMERS C
join ORDERS O on O.CUSTOMER_CODE=C.CUSTOMER_CODE
join ORDER_DETAILS OD on OD.ORDER_NUMBER=O.ORDER_NUMBER
group by C.CUSTOMER_CODE, C.COMPANY, C.PHONE
having count(distinct OD.PRODUCT_REF)= (select count(*) from PRODUCTS);

SELECT DISTINCT C.CUSTOMER_CODE, C.COMPANY, C.PHONE
FROM CUSTOMERS C JOIN ORDERS O ON C.CUSTOMER_CODE = O.CUSTOMER_CODE
JOIN ORDER_DETAILS OD ON O.ORDER_NUMBER = OD.ORDER_NUMBER
JOIN PRODUCTS P ON OD.PRODUCT_REF = P.PRODUCT_REF;

-- Affichez le nombre de commandes passées pour chaque client français. Le tableau obtenu doit afficher les colonnes : code client et nombre de commandes.
SELECT
CUSTOMERS.CUSTOMER_CODE as 'customer code',
COUNT(ORDERS.ORDER_int) AS 'number of orders'
FROM
CUSTOMERS
JOIN ORDERS ON CUSTOMERS.CUSTOMER_CODE = ORDERS.CUSTOMER_CODE
WHERE
CUSTOMERS.COUNTRY = 'France'
GROUP BY
CUSTOMERS.CUSTOMER_CODE;

--Affichez le nombre de commandes passées en 1996, le nombre de commandes passées en 1997 et la différence entre ces deux chiffres. Le tableau obtenu doit contenir les colonnes suivantes : commandes en 1996, commandes en 1997 et différence.
SELECT
(SELECT COUNT(*) FROM ORDERS WHERE YEAR(ORDER_DATE) = 1996) AS ORDERS_1996,
(SELECT COUNT(*) FROM ORDERS WHERE YEAR(ORDER_DATE) = 1997) AS ORDERS_1997,
(SELECT COUNT(*) FROM ORDERS WHERE YEAR(ORDER_DATE) = 1997) -
(SELECT COUNT(*) FROM ORDERS WHERE YEAR(ORDER_DATE) = 1996) AS
DIFFERENCE
from ORDERS;
