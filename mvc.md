# Bonnes conduites MVC

	Cette repo fait un point sur la chronologie à suivre pour ne pas se perdre lors de la création du Model, du Controller et de la View pour une appli Ruby on Rails, de la crétion du dossier de l'appli à l'établissement du MVC.

	Tu y trouveras
	  - les préparatifs avant création du MVC
	  - la création du Model
	  - la crétion du Controller et de sa View



# Préparation


### $ rails new -d postgresql nom_app 
==> création de l'architecture du dossier de ton nom_app

### $ cd nom_app 
==> n'oublie pas de rentrer dans ton nom_app

### Gemfile 
==> mettre tes gem préférées, genre:
  - 'pry'
  - 'table_print'
  - 'faker'
  - décommenter 'bcrypt'

### $ bundle install 
==> installation des gems

### $ rails db:create 
==> il faut bien veiller à créer ta base de données PostgreSQL à chaque fois


# Création du Model

Le Model est chargé de stocker des données dans une table.

### $ rails generate model NomModel nom_column1:type nom_column2:type
(type == string, ou integer, ou boolean, ou text, ou datetime,...)


## 1) Va créer ton Model NomModel dans:

	  nom_app
		|-- app 
		|   |-- models
		|   |   |-- concerns
		|   |   |-- application_record.rb
		|   |   |-- *nom_model.rb* 

Dans ton nom_model.rb:

	class NomModel < ApplicationRecord
	
	   # tu peux rentrer les dépendances de ton model par rapoort aux autres, s'il y en a: 

	    ex: belongs_to :autre_model
	        has_many :autre_models, +/- through: :autre_models, +/- dependent: :destroy

	   # tu peux fixer les règles de ton model avec validates :

	      ex:  validates :nom_column1,
	           presence: true/false,
	           uniqueness: true/false,
	           length: { minimum/maximum: un nombre}
	           numericality: { greater_than: 0 }

	           validates :nom_column2,
	        		{ with: /\A[^@\s]+@([^@\s]+\.)+[^@\s]+\z/, message: "email adress please" }

	       Tu peux auusi créer des méthodes à intégrer aux validates

	    # en cas de mot de passe crypté

	        has_secure_password (en lien avec la gem 'bcrypt')

	    # en cas de callbacks

	      ex:  after_create :send_welcome_email
	           before_save
	           before_validation

	      associer la méthode qui va avec
	           def send_welcome_email
			    		# le code qui envoit l'email
			  		end

	end


## 2) Va créer, en parallèle, une 'migration' du model == table,dans:
	

	nom_app
	|-- db
	|   |-- migrate
	|   |   |-- *numéro_de_migration_create_nom_models.rb*

Dans ta migration:

	class NomModels < ActiveRecord::Migration[5.2]
	  def change
	    create_table :nom_models do |t|
	      t.type :nom_column1 
	      t.type :nom_column2 

	      # tu vas pouvoir y entrer la dépendance de ta table à d'autres, s'il y en a:
	      t.belongs_to: autre_table, index: true  #cette ligne rajoute la référence à la table autre_tables == autre_table_id

	      t.timestamps (crée une column id, created_at et update_at)
	    end
	  end
	end


### $ rails db:migrate
==> va "push" ta table en "up" == va valider les caractéristiques de ta table NomModels


### $ rails db:migrate satus
==> permet de faire un bilan sur l'état de migration de ta ou tes tables: up ou down


### Si besoin: $ rails db:rollback 
==> permet de revenir en arrière en mettant en down ta table pour la modifier
==> il faudra la "push" après avec rails db:migrate


Pour vérifier la gueule de ta table, tu peux aller dans:

	  nom_app
		|-- db
		|   |-- migrate
		|   |   |-- numéro_de_migration_create_nom_models.rb
		|   |   |-- *schema.rb*

WARNING : CONSULTABLE ET NON MODIFIABLE!!!


### $ rails console ou $ rails console --sandbox 
une fois ta/tes tables migrées (rails db:migrate), teste-la/les dans la console
ex: - (+/- tp) n = NomModel.new

	  - (+/- tp) n.nom_column1	

    - (+/- tp) n = NomModel.save

    - (+/- tp) n.autretables (en fonction de tes liens de tables)

    - (+/- tp) n.autretable (en fonction de tes liens de tables)


## Pour une meilleure compréhension de la construction de tes tables par des tiers et éviter de te perdre dans les migrations, il est préférable d'ajouter de nouvelles migrations pour les changements de tables que de faire des $ rails db:rollback


### $ rails generate migration NomMigration 
(ex: AddColumnToNomModel, AddIndexToNomModel)

	  nom_app
		|-- db
		|   |-- migrate
		|   |   |-- numéro_de_migration_create_nom_models.rb
		|   |   |-- *numéro_de_migration_nom_migration.rb*
		|   |   |-- schema.rb

Par exemple, dans ta nouvelle migration, tu pourras ajouter:

	class NomMigration < ActiveRecord::Migration[5.2]
		def change
		   add_column :nom_models, :nom_column, :type
		ou remove_column :nom_models, :nom_column, :type
		ou add_reference :nom_models, :autre_model, foreign_key: true (= rajoute dans la tables nom_models une column autre_model_id de la table autre_models d'un model AutreTable)
		end
	end


Ex de raccourci:
### $ rails genarate migration add_nom_column_to_nom_models nom_column3:type
==> rajoute directement une nom_column3 dans ta table nom_models

N'oublie pas de "pusher" la migration: $ rails db:migrate !!!!

### Utilisation de seeds.rb: pour mettre des entrées (=objets) dans les tables

	  nom_app
		|-- db
		|   |-- migrate
		|   |   |-- numéro_de_migration_create_nom_models.rb
		|   |   |-- numéro_de_migration_nom_migration.rb
		|   |   |-- schema.rb
		|   |   |-- *seeds.rb*

  - soit à la main
  - soit en utilisant la gem 'faker' mise dans le Gemfile

		require 'faker'

		NomModel.destroy_all

		10.times do
			NomModel.create(
				nom_column1: nom_du_Faker (https://github.com/faker-ruby/faker),
				nom_column2: nom_du_Faker
				)
		end

### Afin d'activer le seeds.rb: $ rails db:seed

## ATTENTION: PENSE À CHANGER TON seeds.rb ET FAIRE UN $ rails db:seed A CHAQUE CHANGEMENT DANS TES TABLES!!!!
## FAIS UN MAX DE TESTS EN CONSOLE APRÈS CRÉATION DE TES TABLES HISTOIRE DE NE PAS ÊTRE SURPRIS PAR LA SUITE!!!!


# Création des Controllers et Views

Le controller agit en tant que chef d'orchestre de l'application.
Chaque requête (url) que l'utilisateur envoit au routeur de l'application va appeler le controller correspondant à l'url.
Le Controller consulte son Model relié à sa table de données.
Le Controller récupère les données et les transmet à la View correspondante pour afficher la réponse à la requête.

Chaque Controller est en relation avec son Model et sa View.
Chaque Controller possède:
  - ses méthodes reposant sur le système CRUD == ensemble d'actions que l'on peut faire sur toute application

C = create: créer un contenu (utilisateur, message, mail,...)

R = Read: lire des indormations

U = upadte: ajouter de nouvelles données

D = Destroy: supprimer des données

  - les méthodes sont étroitement liées aux urls envoyées par le routeur au Controller correspondant à travers le système REST

### Le nom du Controlleur sera le même que celui du Model et de la View

### Schéma d'imbrication de CRUD et REST


	   Routes possibles REST                                        |      méthode appelée du nom_controller      |     CRUD

	-----------------------------------------------------------------------------------------------------------------------------------

	get "/nom_controller", to "nom_controller#index"                |               index                         |     READ

	get "/nom_controller/:id", to "nom_controller#show"             |               show                          |     READ

	get "/nom_controller/new", to "nom_controller#new"              |               new                           |     CREATE

	post "/nom_controller", to "nom_controller#create"              |              create                         |     CREATE

	get "/nom_controller/:id/edit", to "nom_controller#edit"        |               edit                          |     UPDATE

	put/patch "/nom_controller/:id", to "nom_controller#update"     |              update                         |     UPDATE

	delete "/nom_controller/:id", to "nom_controller#destroy"       |              destroy                        |     DESTROY


Méthode GET: utilisée pour demander une page (et derrière la renvoyer à l'utilisateur)

Méthode POST: utilisée pour recevoir de l'information d'un utilisateur et derrière créer une ressource en base 

Méthode PUT/PATCH: utilisées pour recevoir de l'information d'un utilisateur et derrière mettre à jour une ressource en base

Méthode DELETE: utilisée pour demander la suppression une ressource en base

### $ rails generate controller nom_controller(+s) index show new create edit update destroy


## 1) Création du Controller avec les méthodes index, show, new, create, edit, update, et destroy

	  nom_app
	  |-- app
		|   |-- controllers
		|   |   |-- concerns
		|   |   |-- application_controller.rb
		|   |   |-- *nom_controllers_controller.rb*


Dans nom_controllers_controller.rb, on retrouve les méthodes annoncées lors de la création du Controller

	class NomControlers Controller < ApplicationController

	  before_action :authenticate_user, only: [:show, :new, :edit, :update, :destroy]
	  before_action :authenticate_current_user, only: [:edit, :destroy]
	  
	  def index (en lien avec la route "/nom_controller", méthode GET)
	    par ex: @nom_models = NomModel.all
	  end

	  def show (en lien avec la route "/nom_controller/:id", méthode GET)
	    généralement: @nom_models = NomModel.find(params[:id])
	                  # récupère l'id de l'url via params
	  end

	  def new (en lien avec la route "/nom_controller/new", méthode GET)
	  	par ex: new user, new article,....
	  end

	  def create (en lien avec la route "/nom_controller", méthode POST)

	    par ex: @nom_model = NomModel.new(nom_column:params[:...], nom_column:.....)

	    if @nom_model.save 
	    	redirect_to gossips_path

			@nom_model.send_welcome_email (# si callbacks dans le model, par ex)

	    else
	    	render 'new'
	    end
	  end


	  def edit
	  end

	  def update
	    par ex: @nom_model = NomModel.find(params[:id])

	      if @nom_model.update(content:params[:content])
	        redirect_to gossip_path
	      else
	        render :edit
	      end
	  end

	  def destroy

	      par ex: @nom_model = NomModel.find(params[:id])

	      if @nom_model.destroy 
	      	redirect_to gossips_path   
	      else
	        render :destroy
	      end
	  end
	end


## 2) Création des routes dans routes.rb


    nom_app
	  |-- config
		|   |-- *routes.rb*

Dans ton routes.rb:

	Rails.application.routes.draw do

	  get "/nom_controller", to "nom_controller#index"
	  get "/nom_controller/:id", to "nom_controller#show"
	  get "/nom_controller/new", to "nom_controller#new" 
	  post "/nom_controller", to "nom_controller#create"  
	  get "/nom_controller/:id/edit", to "nom_controller#edit" 
	  put/patch "/nom_controller/:id", to "nom_controller#update" 
	  delete "/nom_controller/:id", to "nom_controller#destroy"    
		
	end

Les routes peuvent être automatisées
### $ rails resources :nom_controllers

	Rails.application.routes.draw do

		  get "/nom_controller", to "nom_controller#index"
		  get "/nom_controller/:id", to "nom_controller#show"
		  get "/nom_controller/new", to "nom_controller#new" 
		  post "/nom_controller", to "nom_controller#create"  
		  get "/nom_controller/:id/edit", to "nom_controller#edit" 
		  put/patch "/nom_controller/:id", to "nom_controller#update" 
		  delete "/nom_controller/:id", to "nom_controller#destroy" 

		  *resources :nom_controllers*   
			
	end

Toutes les autres routes doivent être supprimées!

	Rails.application.routes.draw do
		  resources :nom_controllers  
	end

On peut ajouter des caractéristiques à resources

	Rails.application.routes.draw do

		  resources :nom_controllers, only:[:create] == le controlleur nom_controller n'appliquera que la méthode create
		  resources :nom_controllers, except:[:create] == le controlleur nom_controller appliquera toutes les méthodes sauf la méthode create

	end

Pour des imbrications de routes == NESTED RESOURCES

	Rails.application.routes.draw do

			  resources :gossips do
			   resources :comments
			  end

	end

Tu peux visualiser tes routes
### $ rails routes

Tu peux visualiser une route spécifique
### $ rails routes|grep nom_controller 


## 3) Création du View correspondant au controller 

	  nom_app
	  |-- app
	  |   |-- views
		|   |   |-- layouts
		|   |   |-- *nom_controller*
		|   |   |   |-- *create.html.erb*
		|   |   |   |-- *destroy.html.erb*
		|   |   |   |-- *edit.html.erb*
		|   |   |   |-- *index.html.erb*
		|	  |   |   |-- *new.html.erb*
		|   |   |   |-- *show.html.erb*
		|   |   |   |-- *update.html.erb*


Chaque fichier .erb correspond à la méthode du Controller associé

Tu mettras dans le fihier .erb:

  - les infos de la page (GET) en html

  - les infos que tu as besoin de récupérer (POST/PUT/PATCH/) via des formulaires

ex:

		<%= form_tag url_for(action: 'create'), method: "post" do %>

		    <%= label_tag 'Contenu' %>
		    <%= text_field_tag 'content'%>
		    <br /> <br />

		    <%= submit_tag "Crée ton gossip" %>
		<% end %>

  - tes liens entre les différentes pages de ton appli

		<%= link_to "clique ici", prefix_path %> == <a href="url_liée_au_path_saisi_dans_ton_link_to">clique ici</a>

"prefix" == prefix de la route vers lequel tu envoies ton utilisateut, visible dans $ rails routes



A NOTER:

  - le dossier layouts contient le fichier application.html.erb, qui est le fichier mère de tous les autres fichiers .erb == permet un html commun à tous les fichiers.erb.

C'est notamment ici qu'il faudra mettre le CDN pour Bootstrap

    nom_app 
		|-- app
		|   |-- views
		|   |   |   |-- *layouts*
		|   |   |   |-- *application.html.erb*


  - tu peux utiliser du langage ruby dans le les fichiers.erb!!! 

en mettant ton code entre <% %> + <% end %>

en mettant <%=  %>, pour afficher le résultat dans ton html 

ex: 

		<% @gossips.each do |gossip| %>

					<%= image_tag("image.jpeg", :size => "319x220") %>

				  <%= link_to "#{gossip.title}", gossip_path(gossip.id)%></h5>
				  <%= gossip.user.first_name %>

		<% end %>


## 4) Crétion des fichiers css correspondants aux fichiers.erb

    nom_app 
	  |-- app
		|   |-- assets 
		|   |   |-- stylesheets
		|   |   |   |-- application.css
		|   |   |   |-- *nom_controller.scss*

Comme pour les views, le fichier application.css est le fihier css mère.




	|-nom_app
	|
	|-- app
	|   |-- assets
	|   |   |-- config
	|   |   |-- images
	|   |   |   |-- .keep
	|   |   |-- javascripts
	|   |   |   |-- channels
	|   |   |   |   -- .keep
	|   |   |   |-- application.js
	|   |   |   |-- cable.js  
	|   |   |-- stylesheets
	|   |   |   |-- application.css
	|   |-- channels
	|   |-- controllers
	|   |   |-- concerns
	|   |   |   |-- .keep
	|   |   |-- application_controller.rb
	|   |-- helpers
	|   |   |-- application_helper.rb
	|   |-- jobs
	|   |-- mailers
	|   |-- models
	|   |   |-- concerns
	|   |   |   |-- .keep
	|   |   |-- application_record.rb
	|   |-- views
	|   |   |-- layouts
	|   |   |   |-- application.html.erb
	|   |   |   |-- mailer.html.environment.rb
	|   |   |   |-- mailers.txt.erb
	|-- bin
	|-- config
	|	|-- environments
	|   |   |-- development.rb
	|   |   |-- production.rb
	|   |   |-- test
	|	|-- initializers
	|   |-- locales
	|   |-- application.rb
	|   |-- boot.rb
	|   |-- cable.yml 
	|   |-- credentials.yml.enc
	|   |-- database.yml
	|   |-- environment.rb
	|   |-- puma.rb
	|   |-- routes.rb
	|   |-- spring.rb
	|   |-- storage.yml
	|-- db
	|   |-- migrate
	|   |-- developent.sqlite3
	|   |-- schema.rb
	|   |-- seeds.rb
	|-- lib
	|-- log
	|-- public
	|-- storage
	|-- test
	|-- tmp
	|-- vendor 
	|-- .gitignore
	|--.ruby-version
	|-- config.ru 
	|-- Gemfile
	|-- Gemfile.lock
	|-- package.json 
	|-- Rakefile
	|-- README.md