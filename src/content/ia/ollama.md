# Ollama

## Instalación

### Con sudo

```
curl -fsSL https://ollama.com/install.sh | sh
```

### Sin sudo

```
mkdir /tmp/ollama/
cd /tmp/ollama/
wget https://github.com/ollama/ollama/releases/download/v0.20.0/ollama-linux-amd64.tar.zst
unzstd -f ollama-linux-amd64.tar.zst
mkdir bin
tar -xvf ollama-linux-amd64.tar -C bin
cd bin
# Start
export LD_LIBRARY_PATH=lib/ollama:$LD_LIBRARY_PATH && bin/ollama serve
# Run (in other terminal)
bin/ollama pull llama3.2:1b
```

## Funcionamiento

```
# Pull model
ollama pull llama3.2:latest
# Install Claude Codex
curl -fsSL https://claude.ai/install.sh | bash
# Run
export ANTHROPIC_AUTH_TOKEN=ollama && export ANTHROPIC_API_KEY="" && export ANTHROPIC_BASE_URL=http://localhost:11434 && claude --model llama3.2:latest
```

Una pregunta sencilla:

```
What is the purpose of the @src/popup/log.js file?
```

Este archivo es super pequeño, tres líneas únicamente:

```
export function reportError(error) {
  console.error(`Error: ${error}`);
}
```

La respuesta tardó 10 minutos y medio. Para probar si un modelo más pequeño es más rápido:

```
ollama pull llama3.2:1b
export ANTHROPIC_AUTH_TOKEN=ollama && export ANTHROPIC_API_KEY="" && export ANTHROPIC_BASE_URL=http://localhost:11434 && claude --model llama3.2:1b
```

Después de estos intentos y ver las 4 CPUs de mi ordenador al 100%, probé a llevarlo a otro ordenador.

Primero modificamos el servicio de Ollama para que el host escuche otras llamadas de la misma red.
```
[Service]
Environment="OLLAMA_HOST=0.0.0.0:11434"
```

Recargamos el servicio:

```
sudo systemctl daemon-reload
sudo systemctl restart ollama
```

Iniciamos Claude, la dirección IP es del host donde cambiamos el servicio:

```
export ANTHROPIC_AUTH_TOKEN=ollama && export ANTHROPIC_API_KEY="" && export ANTHROPIC_BASE_URL=http://192.168.1.39:11434 && claude --model llama3.2:1b
```

## Recursos

- [Acceder a Ollama desde otro host](https://docs.ollama.com/faq#how-can-i-expose-ollama-on-my-network)
- [Configuración Ollama y Claude code](https://docs.ollama.com/integrations/claude-code)
- [Deshabilitar opciones Cloud de Ollama](https://docs.ollama.com/faq#how-do-i-disable-ollama%E2%80%99s-cloud-features)
- [Instalar Ollama sin root](https://github.com/ollama/ollama/issues/7421)
- [Modelos Ollama](https://ollama.com/search)
- [Tutorial Ollama - Real Python](https://realpython.com/ollama/)
