# CI CD con GitHub Actions & ArgoCD
Prueba de construcción de pipeline simple.

# Contexto

Implementé un pipeline CI/CD basado en:

- Docker multi-architecture images
- Kubernetes deployment con Helm charts
- GitHub Actions como CI
- ArgoCD aplicando GitOps para el deploy

El objetivo era automatizar:

- Build de imagen Docker desde GitHub
- Push al registry (Docker Hub)
- Actualización automática del deployment en Kubernetes vía ArgoCD

! Problema detectado !

Durante el deploy en Kubernetes:
Failed to pull image ... no matching manifest for linux/arm64/v8

Esto indicaba que:
- La imagen publicada solo tenía arquitectura amd64
- El cluster (Minikube ARM en macOS Apple Silicon) requería arm64
- Modificar Dockerfile o builds locales no resolvía el problema de forma consistente.

#### Root cause ####
El workflow de GitHub Actions estaba construyendo la imagen sin especificar plataformas:

uses: docker/build-push-action@v5

Por defecto:
solo genera imagen para la arquitectura del runner (amd64).

* Solución aplicada (definitiva) *

Se corrigió el workflow ci.yml agregando:

- name: Build and push Docker image
  uses: docker/build-push-action@v5
  with:
    context: app/
    push: true
    platforms: linux/amd64,linux/arm64
    tags: |
      USER/app:latest
      USER/app:${{ env.BUILD_TAG }}
#########################
Buenas prácticas incorporadas
✔️  Build multi-arch en CI
Permite:

- compatibilidad ARM / AMD
- clusters híbridos
- desarrollo en Apple Silicon sin problemas

✔️  Uso de tags: |
Formato multilinea recomendado porque:

- evita errores de parsing
- facilita versionado (latest + SHA/tag)
- mejora mantenimiento pipeline

✔️  GitOps consistente
Con Helm + ArgoCD:

- cambio en repo → build automático → deploy automático
- infraestructura declarativa
- rollback sencillo.

# Aprendizajes clave

- Multi-arch se controla en CI/CD, no en Dockerfile.
- Helm charts definen qué imagen/tag se deploya.
- ArgoCD sincroniza estado Git con cluster (GitOps).
- Troubleshooting frecuente en ARM clusters locales.


Créditos a Santiago Fernandez.
https://blog.santiagoagustinfernandez.com/un-pipeline-simple-pero-efectivo
