



Rails 4:

## in routes:

get "home/download_pdf"

## in controller(already have pdf):

	def download_pdf
	  send_file(
	    "#{Rails.root}/public/your_file.pdf",
	    filename: "your_custom_file_name.pdf",
	    type: "application/pdf"
	  )
	end

OU 

	def download(file)
	  send_file file.path, :disposition => 'attachment'
	end

## in routes.rb

	<%= link_to 'Download PDF', home_download_pdf_url %>