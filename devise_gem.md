
*************************** gem Devise, choses à faire ***************************

Devise permet de créer un système d'authentification complet en gérerant pour toi :

    L'inscription, avec mot de passe et e-mail sécurisé ;
    La notion de login, logout, session, current_user ;
    La récupération de mot de passe par e-mail ;
    La confirmation de compte par e-mail ;
    Le comptage de login ;
    Le logout après un certain temps d'inactivité ;
    Le blocage de compte après un certain nombre de tentative de login ratées.


## gem 'devise' dans Gemfile

## $ bundle install

## $ rails generate devise:install

==> fichier devise.rb dans /config/initializers

    le fichier de configuration de Devise. On s'en servira par exemple pour paramétrer le service que l'on va utiliser pour les envois d'e-mails

==> fichier devise.en.yml dans /config/locales

    un fichier contenant les messages d'erreur de Devise.

Mettre config.action_mailer.default_url_options = { host: 'localhost', port: 3000 } dans config/environments/develpment.rb

## $ rails g devise nom_model_auquel_va_s'appliquer_devise (généralement user)

Pour dire à Devise quel model va être géré par elle 

==> va créer et modif notamment trois fichiers

### 1) Créer une nouvelle migration

    class AddDeviseToUsers < ActiveRecord::Migration[5.2]

      def self.up
      => ce que devise va ajouter à la table en rails db:migrate

        change_table :users do |t|
          ## Database authenticatable
          Si un model User existe déjà, AVEC des entrées similaires, genre :email et :encrypted_password
          ==> supprimer ces colonnes

          ## Recoverable
          t.string   :reset_password_token
          t.datetime :reset_password_sent_at

          ## Rememberable
          t.datetime :remember_created_at

          ## Trackable
          # t.integer  :sign_in_count, default: 0, null: false
          # t.datetime :current_sign_in_at
          # t.datetime :last_sign_in_at
          # t.inet     :current_sign_in_ip
          # t.inet     :last_sign_in_ip

          ## Confirmable
          # t.string   :confirmation_token
          # t.datetime :confirmed_at
          # t.datetime :confirmation_sent_at
          # t.string   :unconfirmed_email # Only if using reconfirmable

          ## Lockable
          # t.integer  :failed_attempts, default: 0, null: false # Only if lock strategy is :failed_attempts
          # t.string   :unlock_token # Only if unlock strategy is :email or :both
          # t.datetime :locked_at


          # Uncomment below if timestamps were not included in your original model.
          # t.timestamps null: false
        end

        add_index :users, :email,                unique: true
        add_index :users, :reset_password_token, unique: true
        # add_index :users, :confirmation_token,   unique: true
        # add_index :users, :unlock_token,         unique: true
      end

      def self.down
      => ce que devise devra enlever en rollback, cad ce qu'il ajoute!!
        # By default, we don't want to make any assumption about how to roll back a migration when your
        # model already existed. Please edit below which fields you would like to remove in this migration.

        remove_column :users, :reset_password_token
        remove_column :users, :reset_password_sent_at

        remove_column :users, :remember_created_at

        remove_index :users, :email
        begin
        remove_index :users, :reset_password_token
        rescue
        end
        # raise ActiveRecord::IrreversibleMigration
      end
    end

### 2) Modifier le model auquel Devise s'applique

    class User < ApplicationRecord
      # Include default devise modules. Others available are:
      # :confirmable, :lockable, :timeoutable, :trackable and :omniauthable
      devise :database_authenticatable, :registerable,
             :recoverable, :rememberable, :validatable
    end

Ces quelques lignes vont juste dire à ton model, "hey, c'est possible de faire, sur tout objet de type User, des actions de type database_authenticatable, registerable, recoverable, rememberable, et validatable".

### 3) Création de nouvelles routes

ATTENTION: pas de controllers devise user généré....

# ATTENTION: dans routes.rb, mettre les routes devise en premier, sinon les routes des autres controllers (notamment users) vont croiser celles de devise!!!!!

Par ex:

    Rails.application.routes.draw do

      devise_for :users

      resources :users

    end

En faisant $ rails routes

              Prefix Verb   URI     Pattern                                                                                 Controller#Action
           new_user_session GET    /users/sign_in(.:format)                                                                 devise/sessions#new
               user_session POST   /users/sign_in(.:format)                                                                 devise/sessions#create
       destroy_user_session DELETE /users/sign_out(.:format)                                                                devise/sessions#destroy
          new_user_password GET    /users/password/new(.:format)                                                            devise/passwords#new
         edit_user_password GET    /users/password/edit(.:format)                                                           devise/passwords#edit
              user_password PATCH  /users/password(.:format)                                                                devise/passwords#update
                            PUT    /users/password(.:format)                                                                devise/passwords#update
                            POST   /users/password(.:format)                                                                devise/passwords#create
    cancel_user_registration GET   /users/cancel(.:format)                                                                  devise/registrations#cancel
      new_user_registration GET    /users/sign_up(.:format)                                                                 devise/registrations#new
     edit_user_registration GET    /users/edit(.:format)                                                                    devise/registrations#edit
          user_registration PATCH  /users(.:format)                                                                         devise/registrations#update
                            PUT    /users(.:format)                                                                         devise/registrations#update
                            DELETE /users(.:format)                                                                         devise/registrations#destroy
                            POST   /users(.:format)                                                                         devise/registrations#create

## $ rails generate devise:views

Mettre les liens des views

# Devise utilise des helpers importants:

- user_signed_in? == l'utilisateur est-il connecté?
=> s'utilise en condition d'accès à certaine page: "si l'utilisateur est connecté..."

- current_user
=> le user devient current_user quand l'utilisateur est connecté

- athenticate_user!
=> s'utilise dans les callbacks

Pimpage de formulaires: ex

    <div class="container">
      <div class="row">
        <div class="col-md-6 offset-md-3">
          <br><br><br>
          <%= form_for resource, as: resource_name, url: registration_path(resource_name), html: { class: "form-signin mt-3" } do |f| %>
            <h1 class="h3 mb-3 font-weight-normal text-center">Sign up</h1>
            <%= devise_error_messages! %>
            <div class="form-group">
              <%= f.label :email, "Email" %><br />
              <%= f.email_field :email, autofocus: true, autocomplete: "email", class: "form-control" %>
            </div>
            <div class="form-group">
              <%= f.label :password %>
              <% if @minimum_password_length %>
              <em>(<%= @minimum_password_length %> characters minimum)</em>
              <% end %><br />
              <%= f.password_field :password, autocomplete: "new-password", class: "form-control" %>
            </div>
            <div class="form-group">
              <%= f.label :password_confirmation %><br />
              <%= f.password_field :password_confirmation, autocomplete: "new-password", class: "form-control" %>
            </div>
            <div class="actions mt-5">
              <%= f.submit "Sign up", class: "btn btn-lg btn-primary btn-block" %>
            </div>
          <% end %>
          <%= render "devise/shared/links" %>
        </div>
      </div>
    </div>



# Mettre dans environments/develepment config.action_mailer.default_url_options = { :host => 'YOURAPPNAME.herokuapp.com' }

## AVOIR COMMITÉ!!!!

## $ git push heroku master

## $ heroku run rails db:create

## $ heroku run rails db:migrate

## $ heroku run rails db:seed

# ATTENTION, POUR RENDRE LES IMAGES DANS HEROKU:

==> dans config/environments/production.rb 

Mettre "config.assets.compile = false" en "true"