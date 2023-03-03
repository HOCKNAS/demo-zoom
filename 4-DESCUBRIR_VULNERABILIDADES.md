Instalar Checkov:

    brew install checkov

Realizar el escaneo de vulnerabilidades:

    checkov -d . --compact -o json --output-file-path ./

Se ejecuta el script que lee el archivo resultante del paso anterior: results_json.json, e informa de las vulnerabilidades en Slack, ofreciendo una solucion:

    python3 app.py
