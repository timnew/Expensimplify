require 'yaml'
require 'csv'

class String
  def underscore
    word = self.dup
    word.gsub!(/::/, '/')
    word.gsub!(/([A-Z\d]+)([A-Z][a-z])/, '\1_\2')
    word.gsub!(/([a-z\d])([A-Z])/, '\1_\2')
    word.tr!('-', '_')
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

class Expensimplify
  def initialize(input_file, report_file)
    input = YAML.load_file input_file
    @csv = CSV.open report_file, 'wb', quote_char: '"', force_quotes: true
    @csv << %w(timestamp merchant amount)

    @reference_date = Date.today

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

  def today(content)
    @reference_date = case content
                        when String
                          Date.parse content
                        when Date
                          content
                        when :today
                          Date.today
                        else
                          raise "Invalid reference date #{content}"
                      end
  end

  alias_method :reference_date, :today

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

  DATE_DAY_REGEXP = /^[[:blank:]]*\d+[[:blank:]]*$/
  DATE_MONTH_DAY_REGEXP = /^[[:blank:]]*(\d+)[[:blank:]]*[\/-][[:blank:]]*(\d+)[[:blank:]]*$/

  def parse_date(date)
    date = date.to_s

    if date =~ DATE_DAY_REGEXP
      Date.new(@reference_date.year, @reference_date.month, date.to_i)
    elsif match_date = DATE_MONTH_DAY_REGEXP.match(date)
      month, day = match_date.captures
      Date.new(@reference_date.year, month.to_i, day.to_i)
    else
      Date.parse(date)
    end
  end

  def taxi(rows)
    general_generate 'Taxi', rows
  end

  def per_diem(content)
    case content
      when Array
        content.each { |section| per_diem(section) }
      when Hash
        content.symbolize_keys!

        from_date = parse_date content[:from]
        to_date = parse_date content[:to]

        (from_date..to_date).each do |date|
          write_per_diem(date)
        end
    end
  end

  def write_per_diem(date)
    @csv << [date.to_s, :'Travel Payment', 150.to_s]
    @csv << [date.to_s, :'Travel Weekend Payment', 200.to_s] if date.saturday? || date.sunday?
  end
end

task :default, :source_file, :report_file do |_, args|
  Rake::Task[:generate].invoke(*args)
end

def normalize_filename(input, ext, default)
  file_name = if input.nil?
                ''
              else
                input.strip
              end
  
  file_name = "#{default}" if file_name == ''

  file_name = "#{file_name}#{ext}" if File.extname(file_name) == ''

  file_name
end

desc 'Generate report according to yaml file'
task :generate, :source_file, :report_file do |_, args|
  source_file = normalize_filename args.source_file, '.yml', Date.today
  report_file = normalize_filename args.report_file, '.csv', 'report'

  puts "Processing #{source_file}..."

  Expensimplify.new source_file, report_file

  sh "less #{report_file}"
  puts "Report: #{report_file}"
end

desc 'Create a new yaml file and then generate a report from it'
task :new, :file_name do |_, args|
  file_name = normalize_filename args.file_name, '.yml', Date.today

  create_file file_name

  sh "subl -w #{file_name}"

  Rake::Task[:generate].invoke(file_name)
end

def create_file(file_name)
  return if File.exists? file_name

  File.open file_name, 'w' do |file|
    file << <<-END
# Expensimplify Data Sheet
# Taxi:
#   - <date>, <amount list>
# PerDiem:
#   from: <start date>
#   to: <end date>
# PerDiem:
#  -
#    from: <start date 1>
#    to: <end date 2>
#  -
#    from: <start date 2>
#    to: <end date 2>

Today: #{Date.today}
    END
  end
end