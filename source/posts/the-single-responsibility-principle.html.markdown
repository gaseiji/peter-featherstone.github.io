---
title:  "The Single Responsibility Principle"
date:   2017-09-14 16:41:21 +0100
layout: "post"
---

**The** _Single_ Responsibility Principle is the first principle mentioned in the famous SOLID methodology that we should try and follow whenever we build software. Building software in a SOLID way allows us to keep our code flexible, maintainable and most importantly of all – testable!

The Single Responsibility Principle at heart is a very, very simple principle to understand and simply states that:

>A class should have only one reason to change

Simple, right? So why do we constantly see (or code ourselves), code that looks like the example below?

```php	
class AccountingReport {
 
   private $db;
 
   public function __construct() {
      $this->db = new Database;      
   }
 
   public function buildXmlReport() {
      $data = $this->db->getAccountData();
      // Functionality to build XML data and return
   }
 
   public function buildCsvReport() {
      $data = $this->db->getAccountData();
      // Functionality to build CSV data and return
   }
 
   public function buildHtmlTableReport() {
      $data = $this->db->getAccountData();
      // Functionality to build HTML data and return
   }
 
}
```

The class above violates a lot more than just the Single Responsibility Principle but let’s work with it and refactor to a SOLID class design.

First of all, this breaks the Dependency Inversion Principle as we are now completely and utterly dependent upon the concrete Database class, so let’s refactor that out and typehint to an abstraction and as such inverting the dependency and fixing this SOLID violation:

```php
class AccountingReport {
 
   private $db;
 
   public function __construct(DatabaseInterface $db) {
      $this->db = $db;      
   } 
 
   // Rest of methods above removed for brevity
}
```

Much better, but alas! What does an accounting report have to do with a Database? This is where we should realise that the Single Responsibility Principle has been violated as the class will now have to change under the following circumstances:

*Our database interface gets altered by the Database architects to use a different method for getting the data we need.
*Our Api architects need to add a method for returning the data in JSON format.
*Our website designers want to switch the HTML to be returned using tables instead of divs.
*Our accounting department want the CSV table columns altered or a new one added.

As you can see, we already have 4 reasons for this class to change and there are probably more hiding away in there! In addition, this also violates the Open/Closed Principle as if we want to add a new formatting option (such as JSON) then we have to open up this class again and alter it.

So, I hear you ask, what do we do? Well, let’s refactor again. First of all, let’s remove the dependency on a database at all.

Our class should not be interested in a database, it should only be concerned with data and shouldn’t care how it gets this data. For ease of argument we are just going to pass in an array of data but you could pass in different ReportData objects if needed. Our class now looks like the below:

```php
class AccountingReport {
 
   private $data = [];
 
   public function __construct(array $data) {
      $this->data = $data;      
   } 
 
   // Other methods hidden for brevity
}
```

Now our database dependency has gone, but what about the other 3 reasons to change above? This is where the fun begins…

We are going to use something called the Strategy pattern which will enable us to keep our base AccountingReport class SOLID. The Strategy pattern allows us to change the algorithms used by an object at runtime and to switch them in and out at will. To do this, we will create and pass in a Strategy object upon class instantiation.

In our case, the Strategy objects will be represented by different formatting objects. As such, our final base AccountingReport class will look as simple as the below:

```php	
class AccountingReport {
 
   private $data = [];
   private $formatter;
 
   public function __construct(array $data, ReportFormatter $formatter) {
      $this->data = $data;
      $this->formatter = $formatter;      
   } 
 
   public function getFormattedData() {
      return $this->formatter->format($this->data);
   }
 
   public function setFormatter(ReportFormatter $formatter) {
      $this->formatter = $formatter;
   }
 
}
```

That’s it! Our whole class is now reduced to just a constructor, a setFormatter() method to set our formatter at runtime and a getFormattedData() method that returns the data formatted just as we wanted in our initial class setup.

This class now meets our SOLID requirements as we now have no reason to open this class up again, it only really has one reason to change and still meets the Dependency Inversion Princple as all dependencies are now injected.

Now all that’s left do is create our ReportFormatter interface and our different Formatter classes (which are our strategy objects), for example:

```php	
interface ReportFormatter {
   public function format($data);
}

class CsvFormatter implements {
   public function format($data) {
      // Functionality to turn data into CSV format
   }
}
	
class HtmlFormatter implements {
   public function format($data) {
      // Functionality to turn data into XML format
   }
}
```

The good news is that all our formatter classes also follow the SOLID principles, if the web designer wants us to change the Html format we simply need to edit the HtmlFormatter class (and nothing else), similarly for the accountant who wants the CSV data changed slightly we only need to edit the CsvFormatter class.

If we want to create a json format class then we only need to create a new Strategy formatting object and inject it.

Now, let’s see the code example to use it all:
```php
$account_data = [
   'john' => ['salary' => '50000', 'employment_type' => 'full time'],
   'bob' => ['salary' => '25000', 'employment_type' => 'part time']
];
$report = new AccountReport($account_data, new HtmlFormatter);
$html_data = $report->format();
$report->setFormatter(new CsvFormatter);
$csv_data = $report->format();
```

If you have any questions please let me know!
