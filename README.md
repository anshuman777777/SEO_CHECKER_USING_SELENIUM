from selenium import webdriver
from selenium.webdriver.common.by import By
import time
import requests

def check_broken_links(driver):
    links = driver.find_elements(By.TAG_NAME, 'a')
    broken_links = 0
    total_links = 0
    counter = 0
    for link in links:
        if counter > 10:
            break
        counter += 1
        url = link.get_attribute('href')
        if url and url.startswith('http'):
            total_links += 1
            try:
                response = requests.head(url, allow_redirects=True)
                print(response.status_code)
                if response.status_code != 200:
                    broken_links += 1
            except requests.RequestException:
                broken_links += 1
    return broken_links, total_links

def main():
    website_url = input("Please enter your website url: \n")

    chrome_options = webdriver.ChromeOptions()
    chrome_options.add_experimental_option("detach", True)

    driver = webdriver.Chrome(options=chrome_options)

    try:
        start_time = time.time()
        driver.get(website_url)
        load_time = time.time() - start_time

        # Initialize the SEO scores
        scores = {
            'title_tag': 0,
            'meta_description': 0,
            'header_tags': 0,
            'alt_attributes': 0,
            'internal_links': 0,
            'external_links': 0,
            'response_time': 0,
            'broken_links': 0,
        }
        max_scores = {
            'title_tag': 10,
            'meta_description': 10,
            'header_tags': 10,
            'alt_attributes': 10,
            'internal_links': 10,
            'external_links': 10,
            'response_time': 10,
            'broken_links': 10,
        }

        # Check for Title Tag
        try:
            title = driver.find_element(By.TAG_NAME, 'title')
            if title and len(title.get_attribute("innerText")) > 0:
                scores['title_tag'] = 10
        except:
            pass

        # Check for Meta Description
        try:
            meta_description = driver.find_element(By.XPATH, "//meta[@name='description']")
            if meta_description:
                scores['meta_description'] = 10
        except:
            pass

        # Check for Header Tags
        try:
            headers = driver.find_elements(By.XPATH, '//h1 | //h2 | //h3')
            if headers:
                scores['header_tags'] = 10
        except:
            pass

        # Check for Alt Attributes in Images
        try:
            images = driver.find_elements(By.TAG_NAME, 'img')
            for img in images:
                if img.get_attribute('alt'):
                    scores['alt_attributes'] = 10
                    break  # Only need to find one image with alt text for the basic score
        except:
            pass

        # Check for Internal Links
        try:
            internal_links = driver.find_elements(By.XPATH, "//a[contains(@href, 'python.org')]")
            if len(internal_links) > 0:
                scores['internal_links'] = 10
        except:
            pass

        # Check for External Links
        try:
            external_links = driver.find_elements(By.XPATH, "//a[not(contains(@href, 'python.org'))]")
            if len(external_links) > 0:
                scores['external_links'] = 10
        except:
            pass

        # Check for Response Time
        if load_time < 5:  # Arbitrary threshold
            scores['response_time'] = 10
        elif load_time < 8:
            scores['response_time'] = 5

        # Check for Broken Links
        broken_links, total_links = check_broken_links(driver)
        if total_links > 0:
            broken_link_ratio = broken_links / total_links
            if broken_link_ratio == 0:
                scores['broken_links'] = 10
            elif broken_link_ratio < 0.1:
                scores['broken_links'] = 5

        # Print the SEO scores
        print(f"Basic SEO Score for {website_url}:")
        for key, value in scores.items():
            print(f"{key.replace('_', ' ').title()}: {value}/{max_scores[key]}")
        print(f"Response time for the site is: {load_time:.2f} seconds")
        total_score = sum(scores.values())
        max_total_score = sum(max_scores.values())
        print(f"Total SEO Score: {total_score}/{max_total_score}")

    finally:
        driver.quit()

if __name__ == "__main__":
    main()
