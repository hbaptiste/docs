# BackBee Security Component
`BackBee` Security component is based on the security Component of `symfony`. It provides a robust way to secure a web application. Here we will present some of the core concepts of this component : `firewall`, `FirewallMap`, `authentication` , `authorization`, `DecissionAccess`,  `Voter`, `SecurityContext`.

## Overview of security mechanism

The main propose of the security component is to deal with *authentication* and *authorization.* Authentication is a way to grant access to a ressource. When a user is successfully authentified, authorization is a way to control access to a specific ressource.
Authentication is handled by what is called a *firewall*. A application can have many secured areas. A admin area by instance. A secured area is determined by an *url*. When an url that is part of a secured area is requested, behind the hook some registered callbacks are executed. The `Firewall` archives this by listening to the `symfony` `kernel.request` event triggered by the HttpKernel. A `FirewallMap`  that maps a `request` with the registered listeners then checks if the current request is under a secured area or is already secured. A `FirewallMap` Listener can authenticate a request, throws an `AuthenticationException` or do nothing. If one of these listener throws an exception meaning that the request is not authentificated, this exception can either be transformed into an `AccessDeniedHttpException` or used to initiate an authentication process by calling the `Firewall` entry point.

## Authentication
A firewall entry point, as it implements the `authenticationEntryPointInterface`, must provided a `start` method which takes two parameters: a request and the exception previously raised. `start` must returns a `Response` object that can be a page showing a form by instance. If one the FirewallMap Listener can handle the request, it creates a `Token` object that holds a reference to the current user, his role and his credentials. The listener then uses an `authenticationProviderManager` object to authenticate the `Token`. `authenticationProviderManager` will authenticate the token if provided and return an *authenticated* one if the user credential is valid. Otherwise an `AuthenticationException` is raised.
Each *Token* uses a specific type of *authenticationProvider*. An *authenticationProvider* must implements the `authenticationProviderInterface` and provides the two methods :  `authentificate` which takes one paramater that must implements the `TokenInterface` and `supports` which is used to check rather or not the given `token` is supported by the authentication provider .

registered FirewallMap Listeners must implement the `ListenerInterface` thus provided an `handle` method that accept an `GetRequestEvent` object.

## Authorization
Once a user is identified his roles or any other attributes or object of its `token` can be used to determine rather or not he has access to a resource.

### Decision manager
 An `AccessControlManager` is used for making final access decisions based on `Voter`. A voter can decide to `vote` either for grant or deny access to a ressource. A voter can also abstrain itself. By default The `AccessControllerManagers` make his decision according to one these three strategies :
 - **affirmative** (Access is granted if any voter votes for granting access)
 - **consensus** (Access is granted if there are more voters willing to granting access then otherwise
 - **unanimous** (Access is granted if all voters vote for granting access)


#### Understand voter
A voter class must implements the `VoterInterface` which has 4 methods.
-  supportsAttribute($attribute)
-  supportsClass($class)
-  vote(TokentInterface $token, $object, $attributes)

`supportsAttribute` is used to check if the voter supports some attributes of the token.
`supportClass` is user to check it the voter supports a specific class
`vote` handle the vote strategy and must return of those class Constant :

 - VoterInterface::ACCESS_GRANTED
 - VoterInterface::ACCESS_ABSTAIN
 - VoterInterface::ACCESS_DENIED

Bellow is list of Voter used in BackBee
- AccessVoter
- BBAclVoter
- BBRoleVoter
- SudoVoter

## BackBee SecurityContext and the ContextInterface
All the process of creating `Firewall`, `FirewallMap`, `AuthenticationManager`, `AccessDecisionMananger` and listeners is handled by the `BackBee\Security\SecurityContext` object.
However to way of dealing with security Listerners is simplyfied by using a `Context` object.
A security `Context` extends the abstract class `BackBee\Security\Context\BBAuthContext` and defines the **loadListeners** method. `LoadListeners` accepts one parameter which is the security config that can be setted is `repository/Config/security.yml`. A context can be used to instanciate new listener or to register new authentication provider to the `SecuriryContext`.

Bellow we show all the  security contexts provided by BackBee.
 - AnonymousContext
 - BBAuthContext
 - LogoutContext
 - RestfulContext
 - StatelessContext
 - UsernamePasswordContext

#### AnonymousContext
`AnonymouseContext` is used to identify anonymous user.
 **authenticate provider** : `BackBee\Security\Authentication\Provider\AnonymousAuthenticationProvider`

** Listerner **:
`BackBee\Security\Listeners\AnonymousAuthenticationListener`
** Token ** : `BackBee\Security\Token\AnonymousToken`


#### BBAuthContext
`BBAuthContext` is used to authentify a user by using the BackBee User table.
 **authenticate provider** : `BackBee\Security\Authentication\Provider\BBAuthenticationProvider`

** Listerner **: `BackBee\Security\Listeners\AnonymousAuthenticationListener`

** Token ** : `BackBee\Security\Token\BBUserToken`

####LogoutContext
`LogoutContext` handles user logout by removing all setted tokens.
** authenticate provider ** : `none`
** Token ** : `none`
** Listener **: `BackBee\Security\Listeners\LogoutListener`

###RestfulContext
`RestfulContext` is used to authentify rest user with a plublic key.
** authenticate provider **: `BackBee\Security\Listeners\PublicKeyAuthenticationProvider`
** Listener ** : `BackBee\Security\Listeners\PublicKeyAuthenticationListener`
** Token **: `BackBee\Security\Token\BBUserToken`

####StatelessContext
In stateless mode authentication is asked for every requests. `StatelessContext` handles sessions
creation every time stateless mode is disabled.
** authenticate provider **: `none`
** Listener **: `BackBee\Security\Listeners\ContextListener`
** Token **: `none`

#### UsernamePasswordContext
`UsernamePasswordContext`is used to authenticate user by using a login form.
** authenticat provider **: `BackBee\Security\Listeners\Provider\UserAuthenticationProvider`
** Listener **: `BackBee\Security\Listeners\UsernamePasswordAuthenticationListener`
** Token **: `BackBee\Security\Token\UsernamePasswordToken`

## Configuring Security with `security.yml`
Security configurations can be handled with a simple yml file. This file can be found in repository/config/security.yml

```
firewalls:
    bb_area:
        pattern: ^/
        requirements:
            HTTP-X-Requested-With: XMLHttpRequest
            HTTP-X-BB-METHOD: (JsonRpc|Upload)
        provider: bb_user
        bb_auth:
            provider: bb_user
            nonce_dir: security/nonces
            lifetime: 1800
            use_registry: true

providers:
    bb_user:
        entity:
            class: BackBee\Security\User
        secret: %secret_key%
    public_key:
        entity:
            class: BackBee\Security\User

contexts:
    BackBee\Security\Context: [StatelessContext, AnonymousContext, BBAuthContext, UsernamePasswordContext, LogoutContext, RestfulContext]

encoders:
    BackBee\Security\User:
        class: Symfony\Component\Security\Core\Encoder\MessageDigestPasswordEncoder
        arguments:
            algorithm: md5
            encode_as_base64: false
            iterations: 1

acl:
    connection: default

```
