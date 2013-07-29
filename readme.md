Expensimplify
==============

What is Expensimplify
---------------------
Expensimplify means "Expensify" + "Simplify"

[Expensify](www.expensify.com) is a popular SAAS for company to manage expenses.
Expensify provides a very handy features for users to import the expenses from bank recrods, or OCR the receipt with Smart-Scan. 

But what if Expensify doesn't suport your bank, or you are running out of the Smart-Scan credits?

Then you can use Expensimplify to generate the report.


Genearte Report
----------------
Expensify is developed on Ruby and Rake.

To generate a report:

	rake generate[sample.yml, report.csv]

If you're using ZSH, you might need add "\" before brackets
	
	rake generate\[sample.yml, report.csv\]


You can omit the 2nd parameter of the call, then it is default to 'report.csv'.

Create a new datasheet
-----------------------
You can use "new" task to create a new data file and then generate a report against it:

	rake new[data_sheet.yml]

And if you omit the parameter, you will get a report based on the date.

	rake new

Consume the geneated report in Expensify
----------------------------------------

1. Open Expensify
2. Create a new expense
3. Click "A lot"
4. Click "Import Card/Bank"
5. Click "Upload", and find the generated report file
6. For first time use, type "Expensimplify" in "Mapping Name" and "YAML 1.0" in "Account Name" 
7. For the future, you can select "Expensimplify - YAML 1.0" to reuse the mapping
8. Click "Add Expense"
9. Enjoy the uploaded expenses

DataSheet Yaml Sample
---------------------
For more detail, check out the 'sample.yml' file 

### Date
For all the dates used in the data record, Expensimplify supports 3 formats:

(Suppose today is 3013/7/7, then we can express 2013/7/1 as following 3 formats)

* Full Date: 2013/7/1 
* Month and Day: 7/1
* Day: 1

### Amount
For all the amount used in the data record, Expensimplify supports 2 formats:

* Amount: 10.5 => 10.5
* Amount: 10.5 3 16 => 10.5 + 3 + 16 = 29.5

### Taxi
	Taxi: # Taxi Record
	  - 2013/7/1, 13 # => "2013-07-01","Taxi","13.0"
	  - 7/2, 13 	 # => "2013-07-02","Taxi","13.0"
	  - 3, 13 		 # => "2013-07-03","Taxi","13.0"
	  - 4, 12 3 16   # => "2013-07-04","Taxi","31.0"

### Per Diem
	PerDiem: # Per Diem
	  from: 2013/7/1 
	  to:   2013/7/7
 
The record generates

	"2013-07-01","Travel Payment","150"
	"2013-07-02","Travel Payment","150"
	"2013-07-03","Travel Payment","150"
	"2013-07-04","Travel Payment","150"
	"2013-07-05","Travel Payment","150"
	"2013-07-06","Travel Payment","150"
	"2013-07-06","Travel Weekend Payment","200"
	"2013-07-07","Travel Payment","150"
	"2013-07-07","Travel Weekend Payment","200" 
	
If the date range of the Per Diem is not continuing, such as you went back to office for a couple of days, then you can use multi-section per diem record.

	PerDiem: # Multiple Sectioned Record
	  -
	    from: 2013/7/1
	    to:   2013/7/5
	  -
	    from: 2013/7/8
	    to:   2013/7/14

This record will genearte the per diem data from 2013/7/1 to 2013/7/5 and 2013/7/8 to 2013/7/14.	  