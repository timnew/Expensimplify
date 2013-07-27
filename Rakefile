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

desc "Generate report according to yaml file"
task :generate, :source_file, :report_file do |_, args|
  args.with_defaults source_file: 'test.yml', report_file: 'report.csv'
  Processor.new args.source_file, args.report_file
end

desc "Create a new yaml file"
task :new, :file_name do |_, args|
  args.with_defaults file_name: "#{Date.today}.yml"
  
  sh "touch #{args.file_name}"
  sh "mate #{args.file_name}"
end