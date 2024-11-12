# Scraping website

Yang saya perlukan untuk Scraping website adalah sebuah file .avsc untuk schema Avro dan Python Producer beserta BeautifulSoup Library untuk membantu saya Scraping Website.

Berikut adalah file - file yang sudah saya sebutkan tadi:

File .avsc

```
{
  "type": "record",
  "name": "Scraping",
  "fields": [
    {"name": "id", "type": "string"},
    {"name": "title", "type": "string"},
    {"name": "writer", "type": "string"},
    {
      "name": "release_date",
      "type": {"type": "int", "logicalType": "date"}
    },
    {"name": "content", "type": "string"}
  ]
}
```

File Producer Python

```
import time
import requests
from bs4 import BeautifulSoup
from confluent_kafka.avro import AvroProducer
from confluent_kafka.avro.cached_schema_registry_client import CachedSchemaRegistryClient
from avro.schema import parse
import uuid
from datetime import datetime

# Kafka configuration
KAFKA_BROKER = 'server-eric:9092'
TOPIC = 'web_scraping_topic'

# Schema Registry configuration
SCHEMA_REGISTRY_URL = 'http://server-eric:8081'

# Path to the Avro schema file
SCHEMA_PATH = '/home/kafka/confluent-7.7.0/schemas/scraping_schema.avsc'

# Initialize the Schema Registry Client
schema_registry_client = CachedSchemaRegistryClient(SCHEMA_REGISTRY_URL)

# Load the schema from the file
with open(SCHEMA_PATH, 'r') as schema_file:
    schema_str = schema_file.read()

# Parse the schema string using avro.schema.parse()
schema = parse(schema_str)

# Register the schema with the Schema Registry
try:
    schema_id = schema_registry_client.register('scraping_topic-value', schema)
    print(f"Schema registered with ID: {schema_id}")
except Exception as e:
    print(f"Error registering schema: {e}")

# Initialize AvroProducer with the registered schema
avro_producer = AvroProducer({
    'bootstrap.servers': KAFKA_BROKER,
    'schema.registry.url': SCHEMA_REGISTRY_URL
}, default_value_schema=schema)

def send_to_kafka(record):
    try:
        avro_producer.produce(
            topic=TOPIC,
            value=record
        )
        avro_producer.flush()
        print(f"Sent article {record['title']} to Kafka.")
    except Exception as e:
        print(f"Error sending to Kafka: {e}")

def parse_release_date(date_str):
    try:
        date_part = date_str.split("-")[1].strip().split(",")[0]
        date_obj = datetime.strptime(date_part, "%d/%m/%Y")
        epoch_days = (date_obj - datetime(1970, 1, 1)).days
        return epoch_days
    except Exception as e:
        print(f"Error parsing release date: {e}")
        return None

def scrape_article(url, retries=3):
    for attempt in range(retries):
        try:
            response = requests.get(url, timeout=5)
            if response.status_code == 200:
                print(f"Successfully fetched data from article: {url}")
                break  # Exit loop if successful
        except requests.exceptions.ConnectionError as e:
            print(f"Connection error on {url}, attempt {attempt + 1}/{retries}. Retrying...")
            time.sleep(2)  # Wait before retrying
        except requests.exceptions.RequestException as e:
            print(f"Request failed for {url}: {e}")
            return None
    else:
        print(f"Failed to fetch article data after {retries} attempts: {url}")
        return None

    soup = BeautifulSoup(response.text, 'html.parser')

    try:
        title_elem = soup.find('h1', class_='read__title')
        title = title_elem.get_text(strip=True) if title_elem else None
        if not title:
            print(f"Skipping article with no title: {url}")
            return None

        writer_elem = soup.find('div', class_='credit-title-name')
        writer = ', '.join([w.get_text(strip=True) for w in writer_elem.find_all('h6')]) if writer_elem else 'No Writer'

        release_date_elem = soup.find('div', class_='read__time')
        release_date_raw = release_date_elem.get_text(strip=True) if release_date_elem else 'No Date'
        release_date = parse_release_date(release_date_raw) if release_date_raw != 'No Date' else None

        content_elem = soup.find('div', class_='read__content')
        content = content_elem.get_text(strip=True) if content_elem else 'No Content'

        article_id = str(uuid.uuid4())

        record = {
            'id': article_id,
            'title': title,
            'writer': writer,
            'release_date': release_date,
            'content': content
        }

        return record
    except Exception as e:
        print(f"Error scraping article data: {e}")
        return None

def scrape_sites():
    base_urls = [
        "https://pemilu.kompas.com/?source=navbar",
        "https://tekno.kompas.com/?source=navbar",
        "https://ikn.kompas.com/?source=navbar",
        "https://otomotif.kompas.com/?source=navbar",
        "https://bola.kompas.com/?source=navbar",
        "https://lifestyle.kompas.com/?source=navbar",
        "https://lestari.kompas.com/?source=navbar",
        "https://health.kompas.com/?source=navbar",
        "https://money.kompas.com/?source=navbar",
        "https://properti.kompas.com/?source=navbar",
        "https://umkm.kompas.com/?source=navbar",
        "https://edukasi.kompas.com/?source=navbar",
        "https://travel.kompas.com/?source=navbar"
    ]

    article_count = 0

    for base_url in base_urls:
        print(f"Starting scraping on {base_url}...")
        try:
            response = requests.get(base_url, timeout=5)
            response.raise_for_status()
            print(f"Successfully fetched main page of {base_url}")
        except requests.exceptions.RequestException as e:
            print(f"Error accessing {base_url}: {e}")
            continue

        soup = BeautifulSoup(response.text, 'html.parser')
        article_links = soup.find_all('a', href=True)

        for article_link_elem in article_links:
            href = article_link_elem['href']
            if href.startswith('http://') or href.startswith('https://'):
                print(f"Found article link: {href}")
                article_data = scrape_article(href)

                if article_data:
                    send_to_kafka(article_data)
                    article_count += 1

                if article_count >= 1000000:
                    print("Target article count reached.")
                    return

                time.sleep(1)

if __name__ == "__main__":
    scrape_sites()
```

Jika file tersebut sudah disiapkan maka, bisa di run.

Jika memiliki error, maka bisa melakukan revisi code sesuai dengan log error yang diberikan.

Lalu, jika ingin memasukan data Scrape ke database, saya menggunakan JDBC sink yang kemudian data dari scraping website saya tempatkan di database MySQL.

Berikut hasilnya:

![image](https://github.com/user-attachments/assets/fcfb768e-2336-437f-b6fa-d7174311e5a4)

