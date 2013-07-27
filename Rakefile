require 'yaml'
require 'csv'
require 'pry'

class Processor
  def initialize(input_file, report_file)
    input = YAML.load_file input_file
    @csv = CSV.open report_file , 'wb'
    @csv << %w(timestamp merchant amount)

    input.each do |type, items|
      send type.downcase, items
    end

    @csv.close    
  end
  
  def texi(rows)
    rows.map{|row| row.split(',') }.each do |raw_date, amount|
      if raw_date =~ /\d+/ 
        date = Date.new(Date.today.year, Date.today.month, raw_date.to_i).to_s
      elsif raw_date =~ /\d+[\/-]\d+/
        raw_date, month, day = /(\d)+[\/-](\d)/.match raw_date
        date = Date.new(Date.today.year, month, day).to_s
      else
        date = raw_date
      end
    
      amount = amount.split(' ').map{|i| i.to_f}.reduce(:+).to_s
    
      @csv << [date, 'Texi', amount]          
    end
  end
end

task :default, :source_file, :report_file do |_, args|
  Rake::Task[:generate].invoke(*args)
end

task :generate, :source_file, :report_file do |_, args|
  args.with_defaults source_file: 'test.yml', report_file: 'report.csv'
  Processor.new args.source_file, args.report_file
end