Instalar terraform:

    brew tap hashicorp/tap

    brew install hashicorp/tap/terraform

Instalar AIAC:

    brew install gofireflyio/aiac/aiac

Configurar token de OPEN AI:

    export OPENAI_API_KEY=sk-tZYXuDuUHMu6B7TjdTY1T3BlbkFJXXXXXXXXXXXXXX

Generar codigo de terraform:

    aiac get terraform for AWS S3 BUCKET with the name demo-zoom

Guardar el codigo generado de terraform en un archivo `main.tf` y luego ejecutar:

    terrform init

    terrform plan

    terrform apply
    
    Para destruir los recursos desplegados:

    terrform destroy
