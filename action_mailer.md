Rails dispose d'un outil de gestion des envois d'e-mail plutôt bien conçu : Action Mailer. Grâce à lui, tu vas pouvoir automatiser l'envoi de certains e-mails selon les critères que tu définiras (actions de tes utilisateurs, événements ou alertes données, etc..).


L'Action Mailer est organisé en plusieurs éléments au sein d'une app Rails :

    => Des mailers, qui sont ni plus ni moins que des sortes de controllers appliqués aux e-mails. Tout comme les controllers "normaux", ils contiennent des méthodes qui vont faire des appels à la BDD (via les models) et ensuite envoyer des e-mails (au lieu d'envoyer des pages web à des navigateurs).
    => Des views, qui sont des sortes de templates des e-mails à envoyer. Tout comme les views de ton site, elles sont personnalisées grâce à du code Ruby inclus dedans (pour rajouter un nom, un e-mail, le contenu d'un objet récupéré en BDD, etc.). Il existe deux types de views : les .text.erb et les .html.erb car on peut envoyer des e-mails au format HTML comme au format text.

Au final, il faut considérer qu'Action Mailer a un fonctionnement très proche du MVC classique de Rails sauf qu'au lieu d'afficher des pages HTML sur un navigateur, il envoie des fichiers HTML ou text par e-mail.

## $ rails g mailer UserMailer

==> crée un ficher user_mailer.rb dans app/mailers

	class UserMailer < ApplicationMailer
	  default from: 'no-reply@monsite.fr'
	 
	  def welcome_email(user)

	    #on récupère l'instance user pour ensuite pouvoir la passer à la view en @user
	    @user = user 

	    #on définit une variable @url qu'on utilisera dans la view d’e-mail
	    @url  = 'http://monsite.fr/login' 

	    # c'est cet appel à mail() qui permet d'envoyer l’e-mail en définissant destinataire et sujet.
	    mail(to: @user.email, subject: 'Bienvenue chez nous !') 

	  end
	end

==> crée un dossier user_mailer dans aap/views

## Création d'un template d'email welcome_email.html.erb dans app/views/user_mailer/

IMPORTANT: le nom du template est extrêmement important. Il doit être identique à celui de la méthode welcome_email et placé dans le dossier views/user_mailer/ qui contient tous les templates e-mails relatifs au mailer UserMailer

ex: si on veut envoyer un mail de confirmation de réception de mail suite au remplissage un d'un formulaire de contact sur notre site:
- $ rails g mailer UserMailer
- dans app/mailers/user_mailer.rb 

		class UserMailer < ApplicationMailer
		  default from: 'no-reply@monsite.fr'
		 
		  def contact_email(user)

		    #on récupère l'instance user pour ensuite pouvoir la passer à la view en @user
		    @user = user 

		    #on définit une variable @url qu'on utilisera dans la view d’e-mail
		    @url  = 'http://monsite.fr/login' 

		    # c'est cet appel à mail() qui permet d'envoyer l’e-mail en définissant destinataire et sujet.
		    mail(to: @user.email, subject: 'Réception de votre email')

		  end
		end




