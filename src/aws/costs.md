# Costes

## Acceso a la información de coste

A las cuentas que no son la cuenta root, hay que darles permiso, explicado en este [link](https://repost.aws/knowledge-center/iam-billing-access).

## Recibir email mensual de resumen

Podemos activarlo en la root account.

Para activarlo:

Click en nuestro usuario (esquina superior derecha) > Billing and Cost Management > Billing Preferences (menú vertical margen izquierdo).

Debemos activar:

- Invoice delivery preferences > PDF invoices delivery by email.
- Alert preferences:
  - AWS Free Tier alerts.
  - CloudWatch billing alerts.

## Budgets

Podemos activarlo en la root account.

Permite que recibamos una notificación por email cuando el coste excede un valor.

Para crear uno:

Click en nuestro usuario (esquina superior derecha) > Billing and Cost Management > Budgest (menú vertical margen izquierdo) > Create budget.
