# Instalacion-Elasticsearch-KaliPurple

# Instalación stack Elastic

# 1. Instalar dependencias
# ------------------------

sudo apt-get install curl
curl -fsSL https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo gpg --dearmor -o /etc/apt/trusted.gpg.d/elastic-archive-keyring.gpg
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-8.x.list
sudo bash -c "export HOSTNAME=kali-purple.kali.purple; apt-get install elasticsearch -y"

# Anotar el password "token" que se devuelve y que se usará cuando tengamos en local instalado y pida credenciales.


# 2. Convertir a single-node setup:
# -----------------------------------------------------------------------------------------------------
sudo sed -e '/cluster.initial_master_nodes/ s/^#*/#/' -i /etc/elasticsearch/elasticsearch.yml
echo "discovery.type: single-node" | sudo tee -a /etc/elasticsearch/elasticsearch.yml


# 3. Instalar Kibana:
# ------------------
sudo apt install kibana
sudo /usr/share/kibana/bin/kibana-encryption-keys generate -q
# Añadir keys a /etc/kibana/kibana.yml
echo "server.host: \"kali-purple.kali.purple\"" | sudo tee -a /etc/kibana/kibana.yml
# Asegúrese de kali-purple.kali.purple sólo se asigna a "ip suya" en /etc/hosts con el fin de enlazar Kibana a esa interfaz
sudo systemctl enable elasticsearch kibana --now



# 4. Kibana:
# -----------------
sudo /usr/share/elasticsearch/bin/elasticsearch-create-enrollment-token -s kibana

# Abrir en navegador -> http://192.168.253.5:5601
# Introducir user -> elastic and password -> Ver paso 1.
# paste token from above

sudo /usr/share/kibana/bin/kibana-verification-code

# Introducir código de verificación asignado en el navegador.



# 4.Habiliatr HTTPS for Kibana:
# --------------------------

/usr/share/elasticsearch/bin/elasticsearch-certutil ca
/usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12 --dns kali-purple.kali.purple,elastic.kali.purple,kali-purple --out kibana-server.p12
sudo openssl pkcs12 -in /usr/share/elasticsearch/kibana-server.p12 -out /etc/kibana/kibana-server.crt -clcerts -nokeys
sudo openssl pkcs12 -in /usr/share/elasticsearch/kibana-server.p12 -out /etc/kibana/kibana-server.key -nocerts -nodes
sudo chown root:kibana /etc/kibana/kibana-server.key
sudo chown root:kibana /etc/kibana/kibana-server.crt
sudo chmod 660 /etc/kibana/kibana-server.key
sudo chmod 660 /etc/kibana/kibana-server.crt

echo "server.ssl.enabled: true" | sudo tee -a /etc/kibana/kibana.yml
echo "server.ssl.certificate: /etc/kibana/kibana-server.crt" | sudo tee -a /etc/kibana/kibana.yml
echo "server.ssl.key: /etc/kibana/kibana-server.key" | sudo tee -a /etc/kibana/kibana.yml
echo "server.publicBaseUrl: \"https://kali-purple.kali.purple:5601\"" | sudo tee -a /etc/kibana/kibana.yml


# Más información -> https://gitlab.com/kalilinux/kali-purple/documentation/-/tree/main
