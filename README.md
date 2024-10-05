import requests
from bs4 import BeautifulSoup
import pandas as pd
import random
import matplotlib.pyplot as plt

# Step 1: Incorporating proxies
proxies = [
    {'http': 'http://your_proxy_here', 'https': 'https://your_proxy_here'},
    # Add more proxies if needed
]

headers = {
    'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3'
}

# Step 2: Scraping function
def scrape_noon(url, num_products=200):
    products = []
    page = 1

    while len(products) < num_products:
        response = requests.get(f"{url}?page={page}", headers=headers, proxies=random.choice(proxies))
        soup = BeautifulSoup(response.content, 'html.parser')

        # Finding product containers
        product_list = soup.find_all('div', class_='productContainer')  # Update class name based on actual HTML

        for product in product_list:
            try:
                name = product.find('h3', class_='productTitle').text.strip()
                price = float(product.find('div', class_='price').text.strip().replace('AED', '').replace(',', ''))
                brand = product.find('div', class_='brand').text.strip()  # Update based on actual HTML
                seller = product.find('div', class_='seller').text.strip()  # Update based on actual HTML
                
                products.append({
                    'Product Name': name,
                    'Price': price,
                    'Brand': brand,
                    'Seller': seller
                })
                
                if len(products) >= num_products:
                    break
            except:
                continue

        page += 1

    return pd.DataFrame(products)

# Step 3: Saving data to CSV
url = 'https://www.noon.com/uae-en/sports-and-outdoors/exercise-and-fitness/yoga-16328/'
product_data = scrape_noon(url)
product_data.to_csv('noon_products.csv', index=False)

# Step 4: Data Analysis
# Most expensive product
most_expensive = product_data.loc[product_data['Price'].idxmax()]

# Cheapest product
cheapest = product_data.loc[product_data['Price'].idxmin()]

# Number of products by each brand
brand_count = product_data['Brand'].value_counts()

# Number of products by each seller
seller_count = product_data['Seller'].value_counts()

# Print results
print("Most Expensive Product:\n", most_expensive)
print("\nCheapest Product:\n", cheapest)
print("\nNumber of Products by Each Brand:\n", brand_count)
print("\nNumber of Products by Each Seller:\n", seller_count)

# Step 5: Visualization
# Bar chart for number of products by each brand
plt.figure(figsize=(10, 5))
brand_count.plot(kind='bar', color='skyblue')
plt.title('Number of Products by Each Brand')
plt.xlabel('Brand')
plt.ylabel('Number of Products')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()

# Bar chart for number of products by each seller
plt.figure(figsize=(10, 5))
seller_count.plot(kind='bar', color='lightgreen')
plt.title('Number of Products by Each Seller')
plt.xlabel('Seller')
plt.ylabel('Number of Products')
plt.xticks(rotation=45)
plt.tight_layout()
plt.show()

# Histogram for product prices
plt.figure(figsize=(8, 5))
product_data['Price'].plot(kind='hist', bins=20, color='orange')
plt.title('Distribution of Product Prices')
plt.xlabel('Price (AED)')
plt.ylabel('Frequency')
plt.tight_layout()
plt.show()
