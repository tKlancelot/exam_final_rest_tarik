#####################################################################
################ TO DO LIST AUTHENTIFICATION EN JWT #################
#####################################################################


##############################################
#####     METTRE EN PLACE LE PROJET      #####
##############################################

#symfony new exam_final_rest

#composer require api

#composer require --dev maker

#composer require profiler

#php bin/console make:user

#name of controller : [user]

#property to display : [username]

#hash password : yes

#créer une entité ravioles

#configurer la DATABASE_URL dans le fichier .env

#php bin/console doctrine:database:create

#php bin/console make:migration

#php bin/console doctrine:migrations:migrate

#faire des test du crud sur postman

#composer require "lexik/jwt-authentication-bundle"


#à la racine du projet : 
#composer req "lexik/jwt-authentication-bundle"


##############################################
####     CONFIGURER api_platform.yaml     ####
##############################################

```yaml
api_platform:
    allow_plain_identifiers: true
    mapping:
        paths: ['%kernel.project_dir%/src/Entity']
    patch_formats:
        json: ['application/merge-patch+json']
    swagger:
        versions: [3]
```


##############################################
####         GÉNÉRER LES CLÉS SSH         ####
##############################################


```
$ mkdir config/jwt
$ openssl genrsa -out config/jwt/private.pem -aes256 4096
$ openssl rsa -pubout -in config/jwt/private.pem -out config/jwt/public.pem

```

#enter pass phrase => e.g. authenticationProjet

#php bin/console make:controller 

#le nommer : AuthenticationController

#dans mon fichier routes.yaml, spécifier les différentes routes :

```yaml
register:
    path: /register
    controller: App\Controller\AuthenticationController::register
    methods: POST

api:
    path: /api
    controller: App\Controller\AuthenticationController::api

login_check:
    path:     /login_check
    methods:  [POST]
```

##############################################
## CONFIGURER AuthenticationController.php  ##
##############################################

```php
<?php

namespace App\Controller;

use Symfony\Bundle\FrameworkBundle\Controller\AbstractController;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use App\Entity\User;
use Symfony\Component\Security\Core\Encoder\UserPasswordEncoderInterface;

class AuthenticationController extends AbstractController
{
    public function register(Request $request, UserPasswordEncoderInterface $encoder)
    {
//        dump(json_decode($request->getContent()));
//        die();
        $em = $this->getDoctrine()->getManager();
        $username = json_decode($request->getContent())->username;
        $password = json_decode($request->getContent())->password;
        $user = new User();
        $user->setUsername($username);
        $user->setPassword($encoder->encodePassword($user, $password));
        $em->persist($user);
        $em->flush();

        return new Response('User successfully created '. $user->getUsername());
    }

    public function api()
    {
        return new Response(sprintf('Logged in as %s', $this->getUser()->getUsername()));
    }
}
```

#vérifier les nouvelles routes 
#php bin/console debug:router

```
 -------------------------- -------- -------- ------ ------------------------------------- 
  Name                       Method   Scheme   Host   Path                                 
 -------------------------- -------- -------- ------ ------------------------------------- 
  _preview_error             ANY      ANY      ANY    /_error/{code}.{_format}             
  _wdt                       ANY      ANY      ANY    /_wdt/{token}                        
  _profiler_home             ANY      ANY      ANY    /_profiler/                          
  _profiler_search           ANY      ANY      ANY    /_profiler/search                    
  _profiler_search_bar       ANY      ANY      ANY    /_profiler/search_bar                
  _profiler_phpinfo          ANY      ANY      ANY    /_profiler/phpinfo                   
  _profiler_search_results   ANY      ANY      ANY    /_profiler/{token}/search/results    
  _profiler_open_file        ANY      ANY      ANY    /_profiler/open                      
  _profiler                  ANY      ANY      ANY    /_profiler/{token}                   
  _profiler_router           ANY      ANY      ANY    /_profiler/{token}/router            
  _profiler_exception        ANY      ANY      ANY    /_profiler/{token}/exception         
  _profiler_exception_css    ANY      ANY      ANY    /_profiler/{token}/exception.css     
  api_entrypoint             ANY      ANY      ANY    /api/{index}.{_format}               
  api_doc                    ANY      ANY      ANY    /api/docs.{_format}                  
  api_jsonld_context         ANY      ANY      ANY    /api/contexts/{shortName}.{_format}  
  register                   POST     ANY      ANY    /register                            
  api                        ANY      ANY      ANY    /api                                 
  login_check                POST     ANY      ANY    /login_check                         
 -------------------------- -------- -------- ------ ------------------------------------- 
```

##############################################
#####  CONFIGURER packages/security.yaml #####
##############################################

```yaml
security:
    encoders:
        App\Entity\User:
            algorithm: auto

    # https://symfony.com/doc/current/security.html#where-do-users-come-from-user-providers
    providers:
        # used to reload user from session & other features (e.g. switch_user)
        app_user_provider:
            entity:
                class: App\Entity\User
                property: username
    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false

        login:
            pattern:  ^/login
            stateless: true
            anonymous: true
            json_login:
                check_path: /login_check
                success_handler: lexik_jwt_authentication.handler.authentication_success
                failure_handler: lexik_jwt_authentication.handler.authentication_failure

        register:
            pattern:  ^/register
            stateless: true
            anonymous: true

        api:
            pattern:  ^/api
            stateless: true
            anonymous: true
            provider: app_user_provider
            guard:
                authenticators:
                    - lexik_jwt_authentication.jwt_token_authenticator

            # activate different ways to authenticate
            # https://symfony.com/doc/current/security.html#firewalls-authentication

            # https://symfony.com/doc/current/security/impersonating_user.html
            # switch_user: true

    # Easy way to control access for large sections of your site
    # Note: Only the *first* access control that matches will be used
    access_control:
	- { path: ^/login, roles: IS_AUTHENTICATED_ANONYMOUSLY }
	- { path: ^/register, roles: IS_AUTHENTICATED_ANONYMOUSLY }
	- { path: ^/api/ravioles, roles: ROLE_USER }
	- { path: ^/api, roles: IS_AUTHENTICATED_ANONYMOUSLY }


```

#tester le register dans postman
#se mettre en POST, dans body et en JSON

{
	"username": "tarik",
	"password": "password"	
}

#la requête renvoit un 201, élément créé
#User successfully created tarik

#dans l'url http://localhost:8000/login_check
#en post, dans raw et en JSON, passer cet objet en requête

{
	"username": "tarik",
	"password": "password"		
}

#on obtient un long token, avec une durée de validité
#sans les doubles quotes, insérer ce token dans l'onglet Authorization>type:bearer token>Token

#on a maintenant accès à l'api


