# Authentication y authorization

## Authentication vs authorization

- Authentication. Confirma la identidad de un usuario.
- Authorization. Son los permisos que tiene el usuario tras autenticarse.

## 2FA

2FA = Two Factor Authentication

Además del usuario y la contraseña, añadimos otro método de autenticación.

Ejemplo: utilizar un código; por ejemplo que nos envía la web por sms o mail o que lo genera una aplicación que tenemos instalada.

## JWT

JWT = JSON Web Token

Enfocado a autorización.

Sirve para que una web me de información de credenciales sobre el usuario actual.

## MFA

MFA = Multi-factor authentication.

Es como 2FA, significa 2 o más métodos de autenticación.

Por ejemplo, además del código enviado al móvil, por si pierdo el móvil, también puedo usar mi huella dactilar o un dispositivo usb para acceder.

Hay webs en las que tenemos un panel de control y vamos añadiendo los métodos que queramos.

## OAuth

Tiene que ver más con autorización que con autenticación.

Te logeas usando un servidor de terceros.

Por ejemplo, cuando te logeas en una web usando tus credenciales de otra red social como Google, GitHub, Facebook, etc.

## OTP

OTP = One Time Password

Cuando recibes un código (por sms, email, etc.) que copias y pegas en la web para confirmar, por ejemplo, que es tu número de teléfono o dirección de email.

Actualmente, también es utilizado en los magic links. Son links que te da una web y al acceder a ellos quedas logeado sin usar una contraseña.

## SSO

SSO = Single Sign On

Tiene que ver más con autorización que con autenticación.

Tras autenticar a un usuario una sola vez; se hace automáticamente para múltiples lugares. Por ejemplo, te logeas en tu empresa en la web A y puedes acceder a otras webs distintas.
