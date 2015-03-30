---
layout: post
title: "Adding Export to Excel to DocketDonkey"
date: 2015-03-29T23:27:47-04:00
---

I've already got a robust "export to CSV" feature in [DocketDonkey](docketdonkey.com), thanks to [this](http://railscasts.com/episodes/362-exporting-csv-and-excel) excellent RailsCast by Ryan Bates.

``` ruby
class CasesController < ApplicationController
  
  def create
    @cases = Case
        .where("file_date > ? AND file_date < ?", params[:start], params[:complete])
        .order('case_number')

    ...

    respond_to do |format|
      format.csv { 
        send_data @cases.to_csv(public_fields), :filename => 'cases.csv' 
      }
    end

  end

end
```

(The `to_csv` method is on the `case` model.)

Exporting to Excel has been trickier.  For one, I don't have a copy of Excel, so I'm reduced to using Google Sheets as a proxie.      

###Railscast

Thus far I've tried 3 different solutions.  Initially I tried the "write your own XML file" approach dictated by the second half of the previously RailsCast.  For whatever reason this did not work for me, producing a file that Google Sheets thought was XML.

###Axlsx

Next, a combination of [axlsx](https://github.com/randym/axlsx), [axlsx_rails](https://github.com/straydogstudio/axlsx_rails), and [acts_as_xlsx](https://github.com/randym/acts_as_xlsx).  The names alone were deeply confusing; after several fruitless hours (and several `git reset --hard`'s) I shelved the project for a week.  (In fairness to these very popular gems, they're meant for more much heavy-duty Excel manipulations than what I needed.)


###RubyXL

Next I tried using the [RubyXL](https://github.com/weshatheleopard/rubyXL) gem.  I was able to get it working in Pry, in the sense that I could create an .xlsx file that Google Sheets could read.  No luck porting it to the Rails app.  The documentation suggests "The gem can provide StringIO object containing the resulting xlsx file, eliminating the need to save it to disk first. This capability comes in handy for web servers."  All my attempts to do this resulted in an xlsx file containing only the name of the ruby object.  

###Victory

Finally it was in exploring a different gem, [spreadsheet](http://spreadsheet.ch/), that I came across a [tutorial](http://blog.brzezinka.eu/webmaster-tips/rails3/export-to-excel-xls-in-rails3) that revealed my mistake in RubyXL: apparently `StringIO` objects have a `string` method, and `send_data` is expecting some kind of string-ish argument.  I tried it out with RubyXL **it just worked**.

In the `case` model:

```ruby

class Case < ActiveRecord::Base
  require 'rubyXL'
  ...

  def self.to_xls(public_fields)

    workbook = RubyXL::Workbook.new
    sheet = workbook.worksheets[0]

    public_fields.each_with_index { |field, column| sheet.add_cell(0, column, field) }

    all.each.with_index(1) do |kase, row|

      public_fields.each_with_index do |field, column|
        sheet.add_cell(row, column, kase[field])
      end

    end

    workbook.stream.string
  end

end
```

and the new `cases_controller.rb`

```ruby
    ...
    respond_to do |format|
      format.csv { send_data @cases.to_csv(public_fields), :filename => 'cases.csv' }
      format.xls { send_data @cases.to_xls(public_fields), filename: 'cases.xls' }
    end
    ...
```

##Open questions:  

-  should I be returning .xls or .xlsx ?  Does it matter?  

-  How do I test this?

