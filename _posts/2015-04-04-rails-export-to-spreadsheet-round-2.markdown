---
layout: post
title: "Rails Export to Spreadsheet Round 2"
date: 2015-04-04T17:03:51-04:00
---

My previous solution for exporting data from my [DocketDonkey](https://github.com/cfmeyers/DocketDonkey) Rails app was to use the [RubyXL](https://github.com/weshatheleopard/rubyXL) gem.  This worked when N is small, but doesn't scale (in my case, N could very well be 40,000 rows).  

After a brief look at the [xlsx_writer](https://github.com/harvesthq/simple_xlsx_writer) gem (good for creating files on file system but no obvious way to stream from server), I've returned to the [axlsx](https://github.com/randym/axlsx) family of gems.

##Why I didn't want to use axlsx initially

axlsx looked a lot more powerful than what I needed.  All I'm doing is turning an ActiveRecord collection ([Cases](https://github.com/cfmeyers/DocketDonkey/blob/master/app/models/case.rb)) and converting it into an Excel file.  I don't need to mess around with styles, GANTT charts, pivot tables, or even additional worksheets.  

It doesn't help matters that there are actually 3 gems in the family:

-  [axlsx](https://github.com/randym/axlsx) 

-  [axlsx_rails](https://github.com/straydogstudio/axlsx_rails) 

-  [acts_as_xlsx](https://github.com/randym/acts_as_xlsx)

and it's not obvious when you'd need one over the other.

##Differences between the gems

-  [axlsx](https://github.com/randym/axlsx) is the base gem.  In fact, all you need is axlsx.  The other two are helpers. 

-  [acts_as_xlsx](https://github.com/randym/acts_as_xlsx) is a gem that serializes and active record collection to an xlsx format.  It would let me write something like `Case.all.to_xlsx`.

-  [acts_as_xlsx](https://github.com/randym/acts_as_xlsx) is the full on Rails helper.  It lets you extract the "convert to xlsx" logic from the controller into a special view file.


##Step 1:  Try with Pry

Should be a no-brainer, but I only figured this out on my 2nd or 3rd attempt to add the export-to-excel feature.  Of course when you use Pry you're going to be creating a .xlsx *file*, not a stream, but it's a good way to validate the concept.  I took the following code from the [example](https://github.com/randym/axlsx/blob/master/examples/example.rb) from the axlsx repo:


{% highlight ruby %}

require 'axlsx'
p = Axlsx::Package.new
wb = p.workbook

wb.add_worksheet(:name => "Basic Worksheet") do |sheet|
    sheet.add_row ["First Column", "Second", "Third"]
    sheet.add_row [1, 2, 3]
    sheet.add_row ['     preserving whitespace']
end

p.serialize 'test.xlsx'

{% endhighlight %}

This does indeed create a file `test.xlsx` with the requisite cells.  Concept validated!

##Step 2: Add axlsx to Gemfile 

Got error `cannot load such file -- zip/zip (LoadError)`.  [Apparently](https://github.com/randym/axlsx/issues/234) this is due to an incompatability between the latest ruby-zip and axlsx.  It can be fixed by adding `gem 'zip-zip'` to your Gemfile, along with `gem 'axlsx'` and `gem 'axlsx_rails'`.

Also you have to specify `gem 'rubyzip', '~> 1.0.0'`.  It's magic.






