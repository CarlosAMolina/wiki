## Contenidos

- [Reglas existentes](#reglas-existentes)
  - [Mostrar reglas existentes](#mostrar-reglas-existentes)
  - [Ruta reglas](#ruta-reglas)
- [Guardar reglas](#guardar-reglas)
- [Aplicar reglas](#aplicar-reglas)
- [Enlaces](#enlaces)

## Reglas existentes

### Mostrar reglas existentes

```bash
iptables -L -nv --line-number
```

Opciones adicionales:

- `-v`: bytes utilizados

### Ruta reglas

```bash
/etc/sysconfig/iptables
```

## Guardar reglas

```bash
iptables-save > savedrules.txt
```

## Aplicar reglas

```bash
iptables-restore < savedrules.txt
```

## Enlaces

- Tutorial: ver reglas, guardar, aplicar, etc...

<https://www.crybit.com/how-to-save-current-iptables-rules/>