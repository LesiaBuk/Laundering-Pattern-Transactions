## Laundering Pattern Transactions
### Research of synthetic transaction data from IBM for the presence of money laundering indicators
The problem of Money Laundering has always existed everywhere, it has no borders. Over time, with the growing popularity of cryptocurrencies, Money Laundering fraud schemes are becoming more and more complicated.
By definition, Money Laundering is the movement of illicit funds in order to conceal their origin, while creating complex transaction schemes.
Unfortunately, real transaction data is not available to study Money Laundering models (banks cannot provide such data to the public).That is why synthetic transaction data from IBM is used to create a training model of Money Laundering. Financial transactions in this model are conducted via banks, i.e. the payer and receiver both have accounts, with accounts taking multiple forms from checking to credit cards or bitcoin.

This project aims to help you see the chains of transactions more clearly, to delve into where money laundering patterns may begin and end, and to observe the regularities that emerge.<br/>

Here is a link to the Google drive where the project file in pbix format is uploaded:<br/>
https://drive.google.com/drive/folders/1fHbkUUvFiFipzY-X6Vi1wUGcf3ZcLvL9?usp=drive_link <br/>

The following steps were made to create this project:

- Step 1 : Load data into Power BI Desktop, dataset is a csv file.<br/>
- Step2 : In the Power Query Editor, I checked whether the format of all columns was correct, whether there was any blank data in the columns<br/>
- Step 3 : Since the table “HI-Medium_Trans” contains the numbers of banks from which money is sent and those to which money is transferred, in order to calculate the total Laundering Rate, it was decided to perform the following manipulations:<br/>
  - make a duplicate of the “HI-Medium_Trans” table and delete all columns except “From Bank”, “Amount Paid”, “Payment Format”, “Is Laundering”. <br/>
  - Make a duplicate of the “HI-Medium_Trans” table and delete all columns except “To Bank”, “Amount Received”, “Payment Format”, “Is Laundering”.<br/>
  - Filter this table, leaving all the values of the Payment Format column, except for Reinvestment, because these values are already in the previous duplicate with the From Bank column.<br/>
  - Later, using “Append Queries”, these 2 duplicate tables were combined into a table called “Combined_Bank_table”.
- Step 4 : To make the **first dashboard** called ***“Banks in Money Laundering Model”*** in the report view I created a shape, a text box with the dashboard topic, a slicer that shows the date, a slicer that shows whether or not money laundering is taking place (0-no, 1-yes) and information button. Also, for the convenience of user, a Navigation Page was added too.<br/>
- Step 5 : Four card visuals were added to the canvas, one representing the count of unique banks with outgoing transactions (Count of Bank-from), the second card representing the count of unique banks with incoming transactions (Count of to-Banks), the third representing Maximum Laundering Payment made through these banks, and the fourth card representing Average Laundering Payment.<br/>
The formula for Maximum Laundering Payment on the DAX is next:<br/>
```
max_payment = CALCULATE(max('HI-Medium_Trans'[Amount Paid]), 'HI-Medium_Trans'[Is Laundering]=1)
```
The formula for Average Laundering Payment on the DAX is next:<br/>
```
avg_payment = CALCULATE(AVERAGE('HI-Medium_Trans'[Amount Paid]), 'HI-Medium_Trans'[Is Laundering]=1)
```
These visual cards look like this as a result:<br/>
![cards](https://github.com/user-attachments/assets/e268f3d5-ec95-4f6e-9b66-eb309b3667f6)
- Step 6 : Using the Ribbon Chart, I created visualization "Ranked Payment Format of Transactions" , which shows how the payment format changes over time. <br/>

![Ranked_payment_format](https://github.com/user-attachments/assets/0396bd5a-b122-49e5-a68a-a37f94c80271)<br/>

With the help of 100% Stacked Column Chart, a visualization was created to show the percentage of different payment formats during the month
 <br/>
![Stacked_payment_format](https://github.com/user-attachments/assets/fe3d5154-46bb-42a3-94de-ddefef540b41) <br/>
Using bookmarks: it was created an opportunity to see one or another graph when superimposing these 2 charts on each other, changing the bookmarks <br/>
- Step 7 :  Also, using Waterfall Chart, the following visualization was created <br/>

![waterfall_chart](https://github.com/user-attachments/assets/3c5a4272-993a-40da-adac-fb2cd7558b21): <br/>
- Step 8 :Laundering Rate is the ratio of the total number of transactions to the number of transactions that clearly show signs of laundering (column “Is Laundering=1”). In order to ensure that the filters do not affect the Laundering Rate of each bank in the matrix, which will consist of the Bank's indicators, it was decided to create a separate ***calculated table*** with the numbers of banks with outgoing transactions <br/>
```
Bank_table_from = SUMMARIZE('HI-Medium_Trans','HI-Medium_Trans'[From Bank],"Count_of_Transactions",COUNT('HI-Medium_Trans'[From Bank]),"Count_of_Trans_Laund",CALCULATE(COUNT('HI-Medium_Trans'[From Bank]),'HI-Medium_Trans'[Is Laundering]=1), "Sum_of_Amount_Paid",SUM('HI-Medium_Trans'[Amount Paid]))
```
The Dax expression for calculating of Laundering Rate for this table is below:
```
Laundering_Rate = 
VAR rate =
    ROUND(DIVIDE (
        Bank_table_from[Count_of_Transactions],
        Bank_table_from[Count_of_Trans_Laund]
    ),2)
RETURN
     rate 
```
A measure was also created to show total amount, transferred from these banks, and is related to Money Laundering

```
Laundering Amount Paid = CALCULATE(sum('HI-Medium_Trans'[Amount Paid]), 'HI-Medium_Trans'[Is Laundering]=1)
```
So, the following matrix was created: it shows the numbers of banks with outgoing transactions, the date and time of the money transfers, the banks where the money is transferred, the accounts of the people who transferred the money, the number of transactions from these banks, the amount of money paid, the amount of money paid that was related to money laundering, and the Laundering Rate(specifically for from-banks).<br/>
![from_bank](https://github.com/user-attachments/assets/dd4746a6-9995-466b-b654-1787c3c367c3) <br/>

Next, a separate ***calculated table*** had to be created for banks with incoming transactions:
```
Bank_to_table = SUMMARIZE('HI-Medium_Trans','HI-Medium_Trans'[To Bank],"Count_of_Transactions",COUNT('HI-Medium_Trans'[To Bank]),"Count_of_Trans_Laund",CALCULATE(COUNT('HI-Medium_Trans'[To Bank]),'HI-Medium_Trans'[Is Laundering]=1), "Sum_of_Amount_Paid",SUM('HI-Medium_Trans'[Amount Received]))
```
A further measure was also created to calculate the amount of money transferred to the banks that was related to money laundering:

```
Laundering Amount Received = CALCULATE(sum('HI-Medium_Trans'[Amount Received]), 'HI-Medium_Trans'[Is Laundering]=1)
```
The following matrix has been created: it shows the numbers of banks with incoming transactions, the date and time of the money transfers, the banks from which the money comes, the accounts of the people to whom the money is transferred, the number of transactions to this particular banks, the amount of money received, the amount of money received that was related to Money Laundering, and the Laundering Rate (specifically for to-banks).<br/>
![To_bank](https://github.com/user-attachments/assets/e1e95ecb-7687-48d9-8d94-e3844654ac2f)<br/>

The next step was to create ***calculated table*** based on Combined_Bank_table - with aggregated data, i.e. the number of incoming and outgoing transactions for one particular bank, the number of incoming and outgoing transactions related to laundering. The Laundering Rate column was calculated separately, taking into accounting incoming and outgoing transactions in total.
```
Combined_Bank_table_summarize = SUMMARIZE(Combined_Bank_table,Combined_Bank_table[From Bank],"Count_of_Transactions",COUNT(Combined_Bank_table[From Bank]),"Count_of_Trans_Laund",CALCULATE(COUNT(Combined_Bank_table[From Bank]),Combined_Bank_table[Is Laundering]=1), "Sum_of_Amount_Paid",SUM(Combined_Bank_table[Amount Paid]))
```
The following matrix has been created: it shows the numbers of banks with total transactions, count of total transactions, count of total transactions that were related to Money Laundering, and the Laundering Rate <br/>
![combined_banks](https://github.com/user-attachments/assets/e0ef7073-6e8b-4947-9a2e-70282097fa01)<br/>
For the “Laundering Rate” columns in these 3 matrixes, it was decided to make conditional formatting: values below 500 are highlighted with red diamonds to draw attention to the bank with an increased probability of working with Money Laundering.

As a result, the dashboard titled ***“Banks in Money Laundering Model”*** looks like this:<br/>
![dashboard_1_all](https://github.com/user-attachments/assets/865c5055-2d80-4d83-bf3a-9aee1b3b3bdf)<br/>

- Step 9 : The **second dashboard** called ***"Transactions in the Money Laundering Model"*** is intended to provide more information about the transactions: who (which account) transfers money, who (which account) receives money, the relationship between them, etc. <br/>
As in the case of the first dashboard I created a shape, a text box with the dashboard topic, a slicer that shows the date, a slicer that shows whether or not money laundering is taking place, information button. Also button "Back" was added to return to the first page.<br/>
- Step 10 : Six important card visuals were added to the canvas, one card represents the total count of transactions, the second card represents the count of unique Accounts(those who make the transfer,first account), the third represents the amount transferred by Account (“Amount Paid”), the forth card represents the count of unique Account_1 (those who make the transfer, second account), the fifth visual card represents the amount received by Account_1 ("Amount Received"), and the sixth card represents total Launderig Rate (the ratio of the total number of transactions to the number of transactions with Laundering=1).<br/>
As a result, the abovementioned visual cards look like this:<br/>
![cards2](https://github.com/user-attachments/assets/47e9f5c1-435a-4edc-9250-0006e2fe50e6)
- Step 11 :The next step was to clearly show the number of transactions that are not related to Money Laundering and the number of transactions that are, using a donut chart.<br/>
![donut_chart](https://github.com/user-attachments/assets/a8ab859e-b2f7-419d-8c77-b0d20c75372d)
- Step 12: To see if the payment currency has any influence, it was decided to use a stacked column chart to show which payment amounts are in which currency. Also, a slider was added to this visualization, which allows you to select the payment format of your interest.<br/>
![stacked_column_chart](https://github.com/user-attachments/assets/f56d66b7-dea2-4b4f-9eec-132fd6129cd3)<br/>
- Step 13 : In order to provide complete information about accounts that transfer or receive money, it was decided to disclose this with the help of 2 matrixes. The **first matrix** on the bottom left contains a list of accounts that transfer money (“Account”), the date and time of the transfer (“Timestamp”), and the accounts to which the money is transferred (“Account_1”). Values contain data of the number of transactions made from this account (“Count of Transactions”), the amount transferred (“Sum of Amount Paid”) and information whether this account is in the Account_1 column, i.e. not only transfers money, but also receives it.<br/>
In order to do this, the lookupvalue function was used, before it a column containing ones was created in the main table. The formula has the following form ("1" means the presence of Account in Account_1, and “0” means- not).

```
Availability Acc in Account_1 = LOOKUPVALUE('HI-Medium_Trans'[Ones for lookupvalue],'HI-Medium_Trans'[Account_1],'HI-Medium_Trans'[Account],"0")
```
The first matrix has the following view: <br/>
![account_matrix](https://github.com/user-attachments/assets/4e6e8f49-a6e2-45ae-8f70-b46febbefca2)<br/>

Also, a tooltip with 2 graphs was added here: one column chart shows the laundering amount paid and payment formats of it, and the second one shows no laundering amount paid and payment formats of it.<br/>
![tooltip1](https://github.com/user-attachments/assets/7cffcd71-db9a-440c-a6a1-c4e2f6d81e3e)<br/>
A drillthrough feature was added to the “Account” and “Timestamp” rows, showing Count of transactions where Laundering is present, Count of distinct banks from which transactions were made, Count of distinct banks to which transactions were made using 3 visual cards, and some information about the Account`s bank using a table.
![drill-through1](https://github.com/user-attachments/assets/f56575ce-32f7-4ff6-b0af-aa3d369cf887)<br/>
- Step 14 : The **second matrix** on the bottom right contains a list of accounts to where money transfers(“Account_1”), the date and time of the transfer (“Timestamp”), and the accounts from which the money is transferred (“Account”). Values contain data of the number of transactions of this account (“Count of Transactions”), the amount transferred (“Sum of Amount Received”) and information whether this account is present in the Account column, i.e. not only receives money, but also transfers it.The following DAX formula was used for this purpose:
```
Availability Acc_1 in Account = LOOKUPVALUE('HI-Medium_Trans'[Ones for lookupvalue],'HI-Medium_Trans'[Account],'HI-Medium_Trans'[Account_1],"0")
```
The second matrix has the following view: <br/>
![account_1Matrix](https://github.com/user-attachments/assets/b9b740e2-f83d-40cb-9250-0cf8581af9ec)<br/>
A tooltip with 2 graphs was added here: one column chart shows the laundering amount received and payment formats of it, and the second one shows no laundering amount received and payment formats of it.<br/>
![tooltip2](https://github.com/user-attachments/assets/051c8811-9b12-40a2-b39e-e0d4341014be)<br/>
A drillthrough feature was added to the “Account_1” and “Timestamp” rows, showing Count of transactions where Laundering is present, Count of distinct banks to which transactions were made, Count of distinct banks from which transactions were made using 3 visual cards, and some information about the Account_1`s bank using a table.<br/>
![drill-through2](https://github.com/user-attachments/assets/c20e4f2d-7706-4d38-909f-cc94a2aafd52)

As a result, the **second dashboard** titled ***"Transactions in the Money Laundering Model"*** looks like this:
![dashboard_2_all](https://github.com/user-attachments/assets/872d16da-532b-4ce4-95d7-9e02ff39f101)<br/>
- Step 15 : The first dashboard helps us to reveal information about the banks through which money related to money laundering flows, the second dashboard opens up the horizons of accounts that send laundering amounts and accounts that receive these amounts. But how to see the transactional chain of money transfers, how to investigate the pattern of this, because laundering schemes are quite similar to each other. The **third dashboard** entitled ***“How to track down Money Laundering Pattern”*** will help us with this.<br/>
The easiest and most visual way to investigate the patterns of dispersal (when one account transfers an amount of money to several accounts, and these several accounts to each other, so the money is dispersed, or in other words, laundered) or a simple chain of transferring one amount of money from one account to another, from another to another, and so on until it reaches the first account.<br/>
First, let's filter the main table by selecting only transactions with Laundering=1.
```
Table_Account = CALCULATETABLE(FILTER('HI-Medium_Trans', 'HI-Medium_Trans'[Is Laundering]=1))
```
After that, calculated table was visualized with selected columns("Account","From Bank", "Amount Paid", "Payment Currency", "Account_1", "To Bank", "Timestamp", "Payment Format"), next  the “Account” sliser was added, so  you can select the account whose transactions you would like to study.
![table_account](https://github.com/user-attachments/assets/72053dde-f127-4684-9c52-a3de744c445a)<br/>
- Step 16 : In order to see the next links in the chain of transactions of the account selected with the help of the slicer, there is a need to create a calculated table with transactions, in which the column “Account” will contain the value Account_1 from the table “Table_Account”.<br/>
To exclude the problem of circular dependency, let's create a copy of the “Account_table” by filtering out the main transaction table again.
```
Filter_table_main = FILTER('HI-Medium_Trans', 'HI-Medium_Trans'[Is Laundering]=1)
```
Create a table with aggregated data, where GroupBy_Column_Name is the column “Account_1” from the table “Filter_table_main”:
```
List_Account_1 = SUMMARIZE(Table_Account, Table_Account[Account_1],Table_Account[Account],"count_of_banks", count(Table_Account[From Bank]))
```
Now, having a column of unique Account_1 values, we can search for them in the “Account” column of the “Filter_table_main” table
 using the Lookupvalue function:
 ```
 lookup_Account1_1 = LOOKUPVALUE(List_Account_1[count_of_banks],List_Account_1[Account_1],Filter_table_main[Account],0)
 ```
 Next, let's filter the “Filter_table_main” table by taking only those rows where the value Account_1 is present in the “Account” column:
 ```
Copy_filter_table_main1 = FILTER(Filter_table_main, Filter_table_main[lookup_Account1_1]>0)
 ```
 Let`s visualize calculated table "Copy_filter_table_main1" with selected columns("Account","From Bank", "Amount Paid", "Payment Currency", "Account_1", "To Bank", "Timestamp", "Payment Format").

![copy_filter_table_main1](https://github.com/user-attachments/assets/cf91e964-eb9a-43df-9097-b0f32bd3b948) <br/>
 For the interactive component to work, create a Many-to-Many relationship in both directions between the “Filter_table_main” and “Copy_filter_table_main1” tables.<br/>
 - Step 17 : There is a need to create a calculated table with transactions, in which the column “Account” will contain the value Account_1 from the table “Copy_filter_table_main1”. For this create a table with aggregated data, where GroupBy_Column_Name is the column “Account_1” from the table “Copy_filter_table_main1”:
 ```
List_Account_1_2 = SUMMARIZE(CALCULATETABLE(Filter_table_main,Filter_table_main[lookup_Account1_1]<>0),Filter_table_main[Account_1],Filter_table_main[Account],"count_banks", COUNT(Filter_table_main[To Bank]))
 ```
Now, having a column of Account_1 values, we can search for them in the “Account” column of the “Filter_table_main” table
 using the Lookupvalue function:
 ```
 lookup_Account1_2 = LOOKUPVALUE(List_Account_1_2[count_banks],List_Account_1_2[Account_1],Filter_table_main[Account],0)
 ```
 Next, let's filter the “Filter_table_main” table by taking only those rows where the value Account_1 is present in the “Account” column:
 ```
 Copy_filter_table_main2 = FILTER(Filter_table_main, Filter_table_main[lookup_Account1_2]>0)
 ```
Let`s visualize calculated table "Copy_filter_table_main2" with selected columns("Account","From Bank", "Amount Paid", "Payment Currency", "Account_1", "To Bank", "Timestamp", "Payment Format").

![copy_filter_table_main2](https://github.com/user-attachments/assets/cf7aa046-0b88-4dc2-97c3-cfe7847f138d) <br/>
 For the interactive component to work, create a Many-to-Many relationship in both directions between the “Copy_filter_table_main1” and “Copy_filter_table_main2” tables.<br/>

-Step 18 : There is a need to create a calculated table with transactions, in which the column “Account” will contain the value Account_1 from the table “Copy_filter_table_main2”. For this create a table with aggregated data, where GroupBy_Column_Name is the column “Account_1” from the table “Copy_filter_table_main2”:
```
List_Account_1_3 = SUMMARIZE(CALCULATETABLE(Filter_table_main, Filter_table_main[lookup_Account1_2]<>0),Filter_table_main[Account_1],Filter_table_main[Account],"count_banks", COUNT(Filter_table_main[To Bank]), "first_date", min(Filter_table_main[Timestamp]))
```
Having a column of Account_1 values, we can search for them in the “Account” column of the “Filter_table_main” table using the Lookupvalue function:
```
lookup_Account1_3 = LOOKUPVALUE(List_Account_1_3[count_banks],List_Account_1_3[Account_1],Filter_table_main[Account],0)
```
Next, let's filter the “Filter_table_main” table by taking only those rows where the value Account_1 is present in the “Account” column:
 ```
 Copy_filter_table_main3 = FILTER(Filter_table_main, Filter_table_main[lookup_Account1_3]>0)
 ```
Now let`s visualize calculated table "Copy_filter_table_main3" with selected columns("Account","From Bank", "Amount Paid", "Payment Currency", "Account_1", "To Bank", "Timestamp", "Payment Format").

![copy_filter_table_main3](https://github.com/user-attachments/assets/8df00728-6caf-4839-9c39-3f2bc748bec2)<br/>
 For the interactive component to work, create a Many-to-Many relationship in both directions between the “Copy_filter_table_main2” and “Copy_filter_table_main3” tables.<br/>
- Step 19 : The next step is to show the dispersion patterns in these 4 created tables using conditional formatting. ( When one account transfers money to several accounts or vice versa. This increases the probability that a money laundering scheme is involved). <br/>
Let's create 6 measures that will count the repetitions of accounts in the “Account” and “Account_1” columns for each of the 4 visual tables. In order to show this in the visual table, with filters already applied, we will use the allexcept function. The countrows function will help calculate the count.<br/>
```
repeated_Measure0_1 = CALCULATE(COUNTROWS(Table_Account),ALLEXCEPT(Table_Account, Table_Account[Account_1],Table_Account[Account]))
```
```
repeated_Measure1_1 = CALCULATE(COUNTROWS(Copy_filter_table_main1),ALLEXCEPT(Copy_filter_table_main1, Copy_filter_table_main1[Account]))
```
```
repeated_Measure1_2 = CALCULATE(COUNTROWS(Copy_filter_table_main1),ALLEXCEPT(Copy_filter_table_main1, Copy_filter_table_main1[Account_1]))
```
```
repeated_Measure2_1 = CALCULATE(COUNTROWS(Copy_filter_table_main2),ALLEXCEPT(Copy_filter_table_main2, Copy_filter_table_main2[Account]))
```
```
repeated_Measure2_2 = CALCULATE(COUNTROWS(Copy_filter_table_main2),ALLEXCEPT(Copy_filter_table_main2, Copy_filter_table_main2[Account_1]))
```
```
repeated_Measure3_1 = CALCULATE(COUNTROWS(Copy_filter_table_main3),ALLEXCEPT(Copy_filter_table_main3, Copy_filter_table_main3[Account]))
```
```
repeated_Measure3_2 = CALCULATE(COUNTROWS(Copy_filter_table_main3),ALLEXCEPT(Copy_filter_table_main3, Copy_filter_table_main3[Account_1]))
```
Now, using conditional formatting in the columns “Account_1” and “Account” in 4 tables (in the first table only in the column “Account_1”), we set if “Doubled_Account” or “Doubled_Account_1” should be more than 1, in this case, we change the background color to gray.<br/>
As a final result, the **third dashboard** titled ***“How to track down Money Laundering Pattern”*** looks like this:
![dashboard_3_all](https://github.com/user-attachments/assets/4a77f7f0-8879-4956-a65a-a139b99cab95)<br/>
It is very important to check the date: if there is a transaction in this table with an earlier date, it is no longer suitable, and this transaction plays a role in another money laundering chain.<br/>
You can also click on a row in any of these 4 tables, and a visual chain of transactions will be formed, so to speak, "forward" and "backward".
- Step 20 : To the right of the name of the third dashboard, we added an information button that directs to the page with the Instruction for Tracking of Money Laundering Pattern.
![info2](https://github.com/user-attachments/assets/5b7fd512-c168-408a-b40e-d630267aa3bb)
- Step 21 :  In the dashboards “Banks in Money Laundering Model” and “Transactions in the Money Laundering Model”, there are also information buttons that direct the user to an information page with general information about Money Laundering, Laundering Patterns and about icons from conditional formatting.
![info1](https://github.com/user-attachments/assets/03126aed-bcdf-4ff1-9e27-5323b09944f2)


*Thus, the 3 created dashboards will help to see the patterns in the creation of Money Laundering schemes: the dashboard “Banks in Money Laundering Model” helps us to reveal information about the banks through which money related to money laundering flows; the dashboard "Transactions in the Money Laundering Model" helps to identify accounts that send money laundering payments and accounts that receive them; working with the dashboard “How to track down Money Laundering Pattern” is a great interactive opportunity to track either one link or all links in a Money Laundering transaction chain.*


















