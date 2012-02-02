require 'open-uri'
require 'nokogiri'
require 'typhoeus'
require 'json'

task :fetch do
  homepage = Nokogiri::HTML open('https://www.gov.uk/').read
  paths = homepage.xpath("//div[@class = 'browse section']//a/@href").map { |href| href.value.to_s }
  hydra = Typhoeus::Hydra.new
  paths.each do |path|
    section = Typhoeus::Request.new "https://www.gov.uk#{path}", :method => :get
    section.on_complete do |response|
      section_doc = Nokogiri::HTML response.body
      artefact_paths = section_doc.xpath("//div[@class = 'results group']//a/@href").map { |href| href.value.to_s }
      artefact_paths.each do |artefact_path|
        request = Typhoeus::Request.new "https://www.gov.uk#{artefact_path}.json", :method => :get
        request.on_complete do |artefact|
          if response.success?
            begin
              data = JSON.parse artefact.body
              dir = "data/#{data["slug"]}"
              FileUtils.mkdir_p dir
              File.open "#{dir}/data.json", 'w' do |f|
                f.puts artefact.body
              end
              File.open "#{dir}/data.md", 'w' do |f|
                f.puts data["body"]
              end if data.key? "body" 
              puts "#{artefact_path} => #{dir}/"
            rescue => e
              STDERR.puts "Failed to retrieve #{artefact_path}: #{e.message}"
            end
          else
            STDERR.puts "Failed to retrieve #{artefact_path}"
          end
        end
        hydra.queue request
      end
    end
    hydra.queue section
  end
  hydra.run
end
