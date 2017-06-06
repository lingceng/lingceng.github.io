---
layout: post
title: "Export csv or excel by re-using existing HTML view"
date: 2015-12-23 19:34
comments: true
categories: [rails]
---

## The Normal Way
Ruby build-in supports CSV generation as CSV is so simple to compose.
We can convert a 2-D array to a csv string like folllowing:

    require 'csv'
    arr = [['a', 'b', 'c'], ['1', '2', '3']]
    arr.map(&:to_csv).join

And exporting CSV is easy in rails. Let's see the following snippet:

    def index
      @products = Product.order(:name)
      respond_to do |format|
        format.html
        format.csv { send_data @products.to_csv }
      end
    end

See more about export csv in rails [here](http://railscasts.com/episodes/362-exporting-csv-and-excel?view=asciicast)

## I'm lazy
However, I'm a lazy man. I even don't want to prepare 2-D array.

In most cases, we have rendered the table in the view already.
We have done some translations or formats in the view.
And I don't want to move the view code to the controller just for exporting.

## Re-use the view
So, how about re-use the table in the view?
I find out that [render\_to\_string](http://devdocs.io/rails/abstractcontroller/rendering#method-i-render_to_string) can help me do the job.


{% codeblock plain_controller.rb lang:ruby %}
def generate_csv_data(template = nil)
  template ||= "#{controller_name}/#{action_name}.html.slim"
  content = render_to_string(template)
  doc =  Nokogiri::HTML(content)

  table =  doc.at_css('table')
  data = table.css('tr').map do |r|
    r.css('td,th').map(&:text).to_csv
  end.join

  # Convert from utf8 to gbk to make it compatible with Windows Office Excel
  # And Mac number can work with GBK too
  data = data.encode('GBK', undef: :replace, replace: "")
end

# Respond csv file when csv format requested
format.csv { send_data generate_csv_data }
{% endcodeblock %}

Here I get the view page as a string. And extract the table in the string with [nokogiri](http://www.nokogiri.org/).
We can also convert it into an excel easily.
See full codes [here](https://gist.github.com/lingceng/840f97f17128d8a9fd3b)

Inspired by [excelinator](https://github.com/livingsocial/excelinator). But
sadly, the excel exported by excelinator can not be opened with Number on my Mac.
So I write a version to export csv.
