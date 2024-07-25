## Contenidos

- [Escanear](#escanear)
- [Exportar resultados](#exportar-resultados)
- [Errores](#errores)
- [Recursos](#recursos)

## Escanear

Todos los directorios de una carpeta:

```bash
clamscan -r /home
```

Todos los archivos, mostrando el nombre de cada archivo:

```bash
clamscan -r /
```

Todos los archivos, mostrando nombre de archivos infectados y haciendo sonar una alarma al encontrarlos:

```bash
clamscan -r --bell -i /
```

Todos los archivos, mostrando infectados:

```bash
clamscan -r /* --infected
```

## Exportar resultados

Exportar a un archivo:

```bash
 clamscan -r /home/user/Escritorio/ >> analisis.txt
```

Exportar a un log:

```bash
clamscan -r /home/user/Escritorio/ -l prueba.log
```

## Errores

### ERROR: Can't open/parse the config file /usr/local/etc/clamd.conf

Solución:

```bash
sudo dpkg-reconfigure clamav-freshclam
sudo ln -s /etc/clamav/clamd.conf /usr/local/etc/clamd.conf
```

## Recursos

<http://www.clamav.net/>

<https://help.ubuntu.com/community/ClamAV>

<http://www.pc-freak.net/blog/check-infected-files-clamav-log-files/>

<https://askubuntu.com/questions/19699/how-to-view-results-of-last-clamscan-scan/220100#220100>
