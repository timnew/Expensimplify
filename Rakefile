require 'yaml'
require 'csv'

class String
  def underscore
    word = self.dup
    word.gsub!(/::/, '/')
    word.gsub!(/([A-Z\d]+)([A-Z][a-z])/, '\1_\2')
    word.gsub!(/([a-z\d])([A-Z])/, '\1_\2')
    word.tr!("-", "_")
    word.downcase!
    word
  end
end

class Hash
  def symbolize_keys!
    keys.each do |key|
      self[(key.underscore.to_sym rescue key) || key] = delete(key)
    end
    self
  end
end

class Processor
  def initialize(input_file, report_file)
    input = YAML.load_file input_file
    @csv = CSV.open report_file, 'wb', quote_char: '"', force_quotes: true
    @csv << %w(timestamp merchant amount)

    input.each do |type, content|
      method_name = type.underscore
      if self.class.method_defined?(method_name)
        send method_name, content
      else
        general_generate type, content
      end
    end

    @csv.close
  end

  def general_generate(type, rows)
    rows.map { |row| row.split(',') }.each do |date, amount|
      date = parse_date(date)

      amount = parse_amount(amount)

      @csv << [date, type, amount]
    end
  end

  def parse_amount(amount)
    amount.split(' ').map { |i| i.to_f }.reduce(:+).to_s
  end

  def parse_date(date)
    case date
      when Fixnum
        Date.new(Date.today.year, Date.today.month, date)
      when String
        if date =~ /\d+/
          Date.new(Date.today.year, Date.today.month, date.to_i)
        elsif date =~ /\d+[\/-]\d+/
          date, month, day = /(\d)+[\/-](\d)/.match date
          Date.new(Date.today.year, month, day)
        else
          Date.parse(date)
        end
      else
        nil
    end
  end

  def taxi(rows)
    general_generate 'Taxi', rows
  end

  def per_diem(content)
    content.symbolize_keys!

    from_date = parse_date content[:from]
    to_date = parse_date content[:to]

    (from_date..to_date).each do |date|
      @csv << [date.to_s, :'Travel Payment', 150.to_s]
      @csv << [date.to_s, :'Travel Weekend Payment', 200.to_s] if date.saturday? || date.sunday?
    end
  end
end

task :default, :source_file, :report_file do |_, args|
  Rake::Task[:general_generate].invoke(*args)
end

desc "Generate report according to yaml file"
task :general_generate, :source_file, :report_file do |_, args|
  args.with_defaults source_file: 'test.yml', report_file: 'report.csv'
  puts "Processing #{args.source_file}..."
  Processor.new args.source_file, args.report_file
  puts "Report: #{args.report_file}"
end

desc "Create a new yaml file"
task :new, :file_name do |_, args|
  args.with_defaults file_name: "#{Date.today}.yml"
  sh "touch #{args.file_name}"
  sh "mate #{args.file_name}"
  Rake::Task[:general_generate].invoke(args.file_name)
end