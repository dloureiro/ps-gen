require 'fileutils'
require 'etc'

EXECUTABLES=["pdftk","convert","pandoc"]

def get_stdin(message)
	print message
	STDIN.gets.chomp
end

def ask(message, valid_options)
	if valid_options
		answer = get_stdin("#{message} #{valid_options.to_s.gsub(/"/, '').gsub(/, /,'/')} ") while !valid_options.include?(answer)
	else
		answer = get_stdin(message)
	end
	answer
end

# Found here : http://stackoverflow.com/questions/2108727/which-in-ruby-checking-if-program-exists-in-path-from-ruby
def which(cmd)
	exts = ENV['PATHEXT'] ? ENV['PATHEXT'].split(';') : ['']
	ENV['PATH'].split(File::PATH_SEPARATOR).each do |path|
		exts.each { |ext|
			exe = File.join(path, "#{cmd}#{ext}")
			return exe if File.executable? exe
		}
	end
	return nil
end

def verify(verbosity, abortion)
	dependencies_verified = true
	dependencies = {}
	EXECUTABLES.each do |dependency|
		dep = which(dependency)
		if dep != nil then
			puts "#{dependency} : OK" if verbosity
			dependencies[dependency] = true
		else
			puts "#{dependency} : NOK" if verbosity
			dependencies_verified = false
			dependencies[dependency] = false
		end
	end
	if abortion && dependencies_verified == false then
		message = "Il y a des dépendances non satisfaites :\n"
		dependencies_verified.each do |nom, etat|
			if !etat then
				message += "#{nom}"
			end
		end
	end
end

def convertImages(inputDir, outputDir)

	Dir["#{inputDir}/*"].select{|e| File.file?(e) }.each do |file|
		basename = File.basename file, ".graffle"
		filename = File.basename file
		if basename != filename then
			puts "Le fichier graffle ne sera pas converti : #{file}"
		else
			puts "Transfert et conversion du fichier #{filename} vers #{outputDir}"
			`convert -geometry "600x600" #{inputDir}/#{filename} #{outputDir}/#{filename}`
		end
	end
end

task :all => [:epub, :pdf, :html] do

end

task :epub => :verify_data do |task, args|
	verify(false,true)
	mkdir_p 'output/epub', :verbose => false
	puts "Création de la couverture ..."
	Rake::Task[:pdf].invoke(false)
	`pdftk output/pdf/document.pdf cat 1 output output/epub/cover.pdf`
	`convert -quiet output/epub/cover.pdf output/epub/une_cover.png`
	rm 'output/epub/cover.pdf', :verbose => false
	`convert -quiet output/epub/une_cover.png -crop 400x573+90+118 +repage output/epub/cover.png`
	rm 'output/epub/une_cover.png', :verbose => false
	puts "Création des fichiers intermédiaires"

	puts "Création du fichier epub"
	`pandoc -S -s --toc --epub-cover-image=output/epub/cover.png titre.md image_couverture.md quote.md teaser.md document.md -o output/epub/document.epub`
	rm 'output/epub/cover.png', :verbose => false
	puts "Document epub généré dans output/epub"

end

task :teaser do

	`pandoc -f markdown -t latex teaser.md -o teaser.tex`

end

task :quote do

	`pandoc -f markdown -t latex quote.md -o quote.tex`

end

task :image_couverture do

	`pandoc -f markdown -t latex image_couverture.md -o image_couverture.tex`

end

task :pdf, [:verbosity] => [:verify_data,:image_couverture,:quote,:teaser] do |task, args|

	args.with_defaults(:verbosity => true)

	mkdir_p 'output/pdf', :verbose => false

	if args[:verbosity] then

		puts "Creation du document pdf"

	end

	`pandoc -S -s --template=./podcastscience-template.latex --toc -V lang:french -V mainfont:Cambria -V fontsize:11pt -V geometry:a4paper -s --latex-engine xelatex titre.md document.md -o output/pdf/document.pdf`

	rm "image_couverture.tex"
	rm "quote.tex"
	rm "teaser.tex"

end

task :html => :verify_data do

	mkdir_p 'output/html'
	mkdir_p 'output/html/images'
	convertImages "images","output/html/images"
	`pandoc -s teaser.md document.md -o output/html/document.html`

end

task :clean_epub do

	rm_rf "output/epub"

end

task :clean_pdf do

	rm_rf "output/pdf"

end

task :clean_html do

	rm_rf "output/html"

end

task :clean => [:clean_epub, :clean_pdf, :clean_html] do
	rm_rf "output"

end

task :clean_deep => :clean do

	fichiers = ["image_couverture.png","image_couverture.md","document.md", "quote.md", "README.md", "teaser.md", "titre.md"]

	fichiers.each do |file|

		if File.exist?(file)
			if ask("Le fichier #{file} va être supprimé. Souhaitez-vous le supprimer ?", ['y', 'n']) == 'y' then
				rm file	
			end
		end

	end

end

task :generate do

	touch "document.md"
	cp "images/placeholder.png","image_couverture.png"

	titre = ENV["titre"] || "Nouvel épisode"

	if File.exist?("image_couverture.md") then
		if ask("Le fichier image_couverture.md va être écrasé. Souhaitez-vous l'écraser néanmoins", ['y','n']) == 'y' then
			puts "Ecriture du fichier image_couverture.md"
			open("image_couverture.md",'w'){ |f|
				f.puts "\# Crédit Image"
			}
		end
	else
		puts "Ecriture du fichier image_couverture.md"
		open("image_couverture.md",'w'){ |f|
			f.puts "\# Crédit Image"
		}
	end


	utilisateur = Etc.getlogin

	date = (ENV['date'] ? Time.parse(ENV['date']) : Time.now).strftime('%d %B %Y')

	if File.exist?("titre.md") then
		if ask("Le fichier titre.md va être écrasé. Souhaitez-vous l'écraser néanmoins", ['y','n']) == 'y' then
			puts "Ecriture du fichier titre.md"
			open("titre.md", 'w'){ |f|
				f.puts "---"
				f.puts "title: #{titre}"
				f.puts "author: #{utilisateur}"
				f.puts "subtitle: www.podcastscience.fm"
				f.puts "date: #{date}"
				f.puts "---"
			}
		end
	else
		puts "Ecriture du fichier titre.md"
		open("titre.md", 'w'){ |f|
			f.puts "---"
			f.puts "title: #{titre}"
			f.puts "author: #{utilisateur}"
			f.puts "subtitle: www.podcastscience.fm"
			f.puts "date: #{date}"
			f.puts "---"
		}
	end

	if File.exist?("teaser.md") then
		if ask("Le fichier teaser.md va être écrasé. Souhaitez-vous l'écraser néanmoins", ['y','n']) == 'y' then
			puts "Ecriture du fichier teaser.md"
			open("teaser.md", 'w'){ |f|
				f.puts "\# Teaser"
			}
		end
	else
		puts "Ecriture du fichier teaser.md"
		open("teaser.md", 'w'){ |f|
			f.puts "\# Teaser"
		}
	end		

	if File.exist?("quote.md") then
		if ask("Le fichier quote.md va être écrasé. Souhaitez-vous l'écraser néanmoins", ['y','n']) == 'y' then
			puts "Ecriture du fichier quote.md"
			open("quote.md", 'w'){ |f|
				f.puts "\# Quote"
			}
		end
	else
		puts "Ecriture du fichier quote.md"
		open("quote.md", 'w'){ |f|
			f.puts "\# Quote"
		}
	end	

	if File.exist?("README.md") then
		if ask("Le fichier README.md va être écrasé. Souhaitez-vous l'écraser néanmoins", ['y','n']) == 'y' then
			puts "Ecriture du fichier README.md"
			open("README.md", 'w'){ |f|
				f.puts "\# Dossier : #{titre}"
			}

			line_index = 0

			File.readlines("DOCS.md").each do |line|

				if line_index != 0 then
					open("README.md", 'a'){ |f|
						f.puts line
					}
				end
				line_index +=1
			end
		end
	else
		puts "Ecriture du fichier README.md"
		open("README.md", 'w'){ |f|
			f.puts "\# Dossier : #{titre}"
		}

		line_index = 0

		File.readlines("DOCS.md").each do |line|

			if line_index != 0 then
				open("README.md", 'a'){ |f|
					f.puts line
				}
			end
			line_index +=1
		end
	end
end

task :verify do
	verify(true,false)
end

task :verify_data do
	
	fichiers = ["image_couverture.png","image_couverture.md","document.md", "quote.md", "README.md", "teaser.md", "titre.md"]

	fichiers.each do |file|

		abort("Le fichier #{file} n'existe pas. Créez le manuellement ou via la commande rake generate") unless File.exist? file

	end
end

task :default => 'all' 