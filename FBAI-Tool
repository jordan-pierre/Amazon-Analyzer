import requests
import os
import re
from datetime import datetime
import statistics
"""
from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium.webdriver.common.by import By
"""

def main():
    """
    Read ASIN and cost from a CSV file.  Use it to scrape Amazon and generate relevent data points.
    Present the output as a new CSV file.
    """

    """
    Before the loop
    """
    # Current month (for storage fee)
    month = datetime.now().month

    # Set up user agents to prevent IP ban
    # TODO:
    headers = {'User-Agent': 'Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/42.0.2311.90 Safari/537.36'}
    #page = requests.get(url, headers=headers)

    #ua = UserAgent()
    #header = {'User-Agent': str(ua.chrome)}

    # Read HTML from a file I made
    #r = open("B071JPD9M3-page.txt", "r+")
    #productRawHTML = r.read()

    # Read ASIN and Cost from input CSV
    # TODO:

    # Create output CSV file
    # TODO:

    # Store data in an output variable
    data = []

    # Make title row for output CSV
    first_row = ['ASIN', 'Title', 'Profit', 'Margin', 'Rank', 'Category', 'No. Prime Sellers', 'Sold by Amazon', 'BBP',
                 'Cost', 'Fees', 'Reviews', 'Link']

    # Append first row to output variable
    data.append(first_row)

    """
    Loop here down
    """
    #for [row] in [file]:

    # Read the ASIN from the input file
    asin = "B000W5SLB8"
    #asin = "B00X59DBZO"

    # Read the cost from supplier from the input file
    cost = "6"

    # Provide the shipping cost per unit (might deleted)
    shipping = "0"

    # Generate the product and listing pages URL
    productURL = "https://www.amazon.com/dp/{}".format(asin)
    listingsURL = "https://www.amazon.com/gp/offer-listing/{}/" \
                  "ref=dp_olp_new_mbc?ie=UTF8&condition=new&f_primeEligible=true".format(asin)

    print("Product Page = {}".format(productURL))
    print("Listings Page = {}".format(listingsURL))

    # Get the product and listings pages HTML
    productRawHTML = requests.get(productURL).text
    productHTML = os.linesep.join([s for s in productRawHTML.splitlines() if s.strip()]) # Strip the file of empty lines
    listingsHTML = requests.get(listingsURL).text

    """
    Collect data points from the appropriate webpages
    """

    # Title
    title = getTitle(productHTML)
    print("Title = {}".format(title))

    # BBP
    BBP = getBBP(productHTML)
    print("BBP = {}".format(BBP))

    # Num Reviews
    reviews = getReviews(productHTML)
    print("Reviews = {}".format(reviews))

    # Category
    category = getCategory(productHTML)
    print("Category = {}".format(category))

    # Best Sellers Rank
    rank = getBSR(productHTML, category)
    print("Rank = {}".format(rank))

    # Number of Prime Sellers and condition = new
    FBA = getFBASellers(listingsHTML)
    print("Number of prime sellers = {}".format(FBA))

    # Bool sold by Amazon
    amazon = isSoldByAmazon(listingsHTML)
    print("Sold by Amazon = {}".format(amazon))

    # Shipping weight
    shipping_weight = getWeight(productHTML)
    print('Weight = {}'.format(shipping_weight))

    # Dimensions (dimensions on the product page are often different than that on the fee calc)
    dimensions = getDimension(productHTML)
    print("Dimensions = {}".format(dimensions))

    # Size category (not in output)
    sizeCategory = getSizeCat(shipping_weight, dimensions)
    print("Size category = {}".format(sizeCategory))

    # Fulfillment fee (not in output)
    fulfillmentFee = getFBAFees(shipping_weight, sizeCategory)
    print("Fulfillment Fees = {}".format(fulfillmentFee))

    # Storage fee (not in output)
    storageFee = getStorageFees(sizeCategory, dimensions, month)
    print("Storage Fees = {}".format(storageFee))

    # Referral fee (not in output)
    referralFee = getReferralFee(BBP, category)
    print("Referral Fees = {}".format(referralFee))

    # Total fees
    totalFees = sumFees(fulfillmentFee, storageFee, referralFee)

    # Calculate Profit
    cost = float(cost)
    profit = getProfit(BBP, cost, fulfillmentFee, storageFee, referralFee)
    print("Profit = {}".format(profit))

    # Calculate profit margin
    margin = getMargin(profit, BBP)
    print("margin = {}".format(margin))

    # Indicate if errors are from IP ban
    IP_indicator = "Sorry, we just need to make sure you're not a robot."
    if IP_indicator in productHTML:
        print("""
            ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
            ~~~~~~~~~~~~~~IP BANNED~~~~~~~~~~~~~~
            ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
            """)

    # Summarize data
    row = [asin, title, profit, margin, rank, category, FBA, amazon, BBP, cost, totalFees, reviews, productURL]

    # Append row to end of output CSV
    data.append(row)

# -- Functions --
def sumFees(fulfillmentFee, storageFee, referralFee):
    """
    Simply sums the fees to get a total fee per unit, which will be used in the results table. Seperate function to
    try/catch in case one of the individual fees are "N/A".

    :param fulfillmentFee: Amazon fulfillment fee
    :param storageFee: Amazon storage fee
    :param referralFee: Amazon referral fee
    :return: Float sum of all Amazon fees.
    """
    try:
        total = fulfillmentFee + storageFee + referralFee
    except:
        total = "N/A"
    return total
def getMargin(profit, price):
    """
    Calculates profit margin using profit / price * 100.

    :param profit: Price - cost - fees
    :param price: Buy box price
    :return: Profit margin as percent
    """
    try:
        margin = profit / price * 100
    except:
        margin = "N/A"
    return margin

def getProfit(price, cost, fulfillmentFee, storageFee, referralFee):
    """
    Calculates the profit using Buy box price - product cost - fees.

    :param price: BBP
    :param cost: Cost of product from distributor
    :param fulfillmentFee: Amazon fulfillment fee
    :param storageFee: Amazon storage fee
    :param referralFee: Amazon referral fee
    :return: float profit
    """
    try:
        profit = price - cost - fulfillmentFee - storageFee - referralFee
    except:
        profit = "N/A"
    return profit

def getReferralFee(price, cat):
    """
    Gets product referral fee based on the unit price and category.  Some categories are more specific
    than the broad categories used in the estimator.
    Details: https://sellercentral.amazon.com/gp/help/200336920?language=en_US&ref=id_200336920_cont_G19211

    :param price: Buy box price
    :param cat: Broad category of the product
    :return: Referral Fee
    """
    # After Feb 19, 2019, minimum referral fee will decrease from $1.00 to $0.30
    minRef = 0.30 # Minimum referral fee
    closingFee = 1.80 # Closing fee applies to all media product sold (movies, books, videogames)
    if cat == 'Amazon Device Accessories':
        fee = max(0.45 * price, minRef)
    elif cat == 'Baby Products':
        if price > 10:
            fee = max(0.15 * price, minRef)
        else:
            fee = max(0.08 * price, minRef)
    elif cat == 'Books':
        fee = 0.15 * price + closingFee
    elif cat == 'Camera and Photo':
        fee = max(0.08 * price, minRef)
    elif cat == 'Cell Phones & Accessories':
        if price < 100:
            fee = max(0.15 * price, minRef)
        else:
            fee = max(0.08 * price, minRef)
    elif cat == 'Electronics':
        fee = max(0.08 * price, minRef)
    elif cat == 'Movies & TV':
        fee = max(0.08 * price, minRef)
    elif cat == 'Appliances':
        if price > 300:
            fee = max(0.08 * price, minRef)
        else:
            fee = max(0.15 * price, minRef)
    elif cat == 'Office Products':
        fee = max(0.15 * price, minRef)
    elif cat == 'Musical Instruments':
        fee = max(0.15 * price, minRef)
    elif cat == 'Outdoors':
        fee = max(0.15 * price, minRef)
    elif cat == 'Home & Garden' or cat == 'Pet Supplies':
        fee = max(0.15 * price, minRef)
    elif cat == 'Computers & Accessories':
        fee = max(0.06 * price, minRef)
    elif cat == 'Video Games':
        fee = max(0.15 * price, minRef) + closingFee
    elif cat == 'Clothing, Shoes & Jewelry':
        fee = max(0.17 * price, minRef)
    elif cat == 'Tools & Home Improvement':
        fee = max(0.15 * price, minRef) # Technicaly 12% on base powertools
    elif cat == 'Toys & Games':
        fee = max(0.15 * price, minRef)
    elif cat == 'Beauty & Personal Care':
        if price > 10:
            fee = max(0.15 * price, minRef)
        else:
            fee = max(0.08 * price, minRef)
    elif cat == 'Health & Household':
        if price > 10:
            fee = max(0.15 * price, minRef)
        else:
            fee = max(0.08 * price, minRef)
    elif cat == 'Grocery & Gourmet Food':
        if price > 15:
            fee = max(0.15 * price, minRef)
        else:
            fee = max(0.08 * price, minRef)
    elif cat == 'Industrial & Scientific':
        fee = max(0.12 * price, minRef)
    # FIXME: Can't detect
    elif cat == 'Jewelry': # Currently doesn't detect jewelyry but just 'Clothing, Shoes & Jewelry'
        boundaryPrice = 250
        if price > boundaryPrice:
            fee = max(0.05 * (price - boundaryPrice) + boundaryPrice * 0.2, minRef)
        else:
            fee = max(0.2 * price, minRef)
    # FIXME: Can't detect.  'Women's Handbags' is default in search bar (not when /dp/asin)
    elif cat == 'Shoes, Handbags & Sunglasses': # Currently doesn't detect any but just 'Clothing, Shoes & Jewelry'
        if price > 75:
            fee = max(0.18 * price, minRef)
        else:
            fee = max(0.15 * price, minRef)
    # FIXME: Can't detect.
    elif cat == 'Watches':  # Currently doesn't detect 'Watch' but just 'Clothing, Shoes & Jewelry'
        boundaryPrice = 1500
        if price > boundaryPrice:
            fee = max(0.03 * (price - boundaryPrice) + boundaryPrice * 0.16, minRef)
        else:
            fee = max(0.16 * price, minRef)
    # FIXME: ADD CATEGORIES
    else:
        fee = "N/A"
    return fee

def getStorageFees(size, dim, month):
    """
    Calculates referral fees based on product size and dimensions and the month of the year.
    Details: https://sellercentral.amazon.com/gp/help/200336920?language=en_US&ref=id_200336920_cont_G19211

    :param size: Size category (Small/Med/Large + Standard/Oversized)
    :param dim: Product dimensions
    :param month: Current month
    :return:
    """
    # RATES PER CUBIC FT
    STANDARD_BEFORE_OCT = 0.69
    OVERSIZED_BEFORE_OCT = 0.48
    STANDARD_AFTER_OCT = 2.40
    OVERSIZED_AFTER_OCT = 1.20
    # Cubic inches in cubic foot
    IN3_in_FT3 = 1728

    try:
        # Volume in cubic feet
        vol = dim[0] * dim[1] * dim[2] / IN3_in_FT3

        if size in ['small_oversized', 'medium_oversized', 'large_oversized', 'special_oversized']:
            oversized = True
        else:
            oversized = False

        # Jan-Sept Rate
        if month <= 9:
            if not oversized:
                fee = vol * STANDARD_BEFORE_OCT
            else:
                fee = vol * OVERSIZED_BEFORE_OCT
        # Oct-Dec Rate
        else:
            if not oversized:
                fee = vol * STANDARD_AFTER_OCT
            else:
                fee = vol * OVERSIZED_AFTER_OCT
    except:
        fee = "N/A"
    return fee

def getFBAFees(weight, size):
    """
    Calculates fulfillment fees based on product weight and size category.
    Details: https://services.amazon.com/fulfillment-by-amazon/pricing.html

    :param weight: Product weight
    :param size: Product size category
    :return: Fulfillment fee
    """
    try:
        if size in ['small_oversized', 'medium_oversized', 'large_oversized', 'special_oversized']:
            oversized = True
        else:
            oversized = False

        if size == 'small_standard' and weight <= 1:
            fee = 2.41
        elif not oversized and weight <= 1 and weight > 0:
            fee = 3.19
        elif not oversized and weight <= 2 and weight > 1:
            fee = 4.71
        elif not oversized and weight >= 2:
            fee = 4.71 + 0.38*(weight - 2)
        elif size == 'small_oversized':
            if weight >= 2:
                fee = 8.13 + 0.38 * (weight - 2)
            else:
                fee = 8.13
        elif size == 'medium_oversized':
            if weight >= 2:
                fee = 9.44 + 0.38 * (weight - 2)
            else:
                fee = 9.44
        elif size == 'large_oversized':
            if weight >= 90:
                fee = 73.18 + 0.79 * (weight - 90)
            else:
                fee = 73.18
        elif size == 'special_oversized':
            if weight >= 90:
                fee = 137.32 + 0.91 * (weight - 90)
            else:
                fee = 137.32
        else:
            fee = "N/A"
    except:
        fee = "N/A"
    return fee

def getSizeCat(weight, dim):
    """
    Gets size category of the product based on its weight and dimensions.
    Details: https://services.amazon.com/fulfillment-by-amazon/pricing.html

    :param weight: product weight
    :param dim: product dimensions
    :return: size category
    """
    maxSide = max(dim)
    minSide = min(dim)
    median = statistics.median(dim)
    lg = 2*(min(dim) + median) + max(dim) # longest side plus girth calculation
    # Small Standard
    SS_WEIGHT_LIMIT = 0.75
    SS_MAX_MAX_DIM = 15
    SS_MAX_MIN_DIM = 0.75
    SS_MAX_MED_DIM = 12
    # Large Standard
    LS_WEIGHT_LIMIT = 20
    LS_MAX_MAX_DIM = 18
    LS_MAX_MIN_DIM = 8
    LS_MAX_MED_DIM = 14
    # Small Oversized
    SO_WEIGHT_LIMIT = 70
    SO_MAX_MAX_DIM = 60
    SO_MAX_MED_DIM = 30
    SO_MAX_LONGEST_SIDE_PLUS_GIRTH = 130
    # Medium Oversized
    MO_WEIGHT_LIMIT = 150
    MO_MAX_MAX_DIM = 108
    MO_MAX_LONGEST_SIDE_PLUS_GIRTH = 130
    # Large Oversized
    LO_WEIGHT_LIMIT = 150
    LO_MAX_MAX_DIM = 108
    LO_MAX_LONGEST_SIDE_PLUS_GIRTH = 165
    # Special Oversized
    SpO_WEIGHT_LIMIT = 150
    SpO_LONGEST_SIZE = 108
    SpO_LONGEST_SIZE_PLUS_GIRTH = 165
    try:
        if weight <= SS_WEIGHT_LIMIT and maxSide <= SS_MAX_MAX_DIM and minSide <= SS_MAX_MIN_DIM and median <= SS_MAX_MED_DIM:
            size = 'small_standard'
        elif weight <= LS_WEIGHT_LIMIT and maxSide <= LS_MAX_MAX_DIM and minSide <= LS_MAX_MIN_DIM and median <= LS_MAX_MED_DIM:
            size = 'large_standard'
        elif weight <= SO_WEIGHT_LIMIT and maxSide <= SO_MAX_MAX_DIM and median <= SO_MAX_MED_DIM and lg <= SO_MAX_LONGEST_SIDE_PLUS_GIRTH:
            size = 'small_oversized'
        elif weight <= MO_WEIGHT_LIMIT and maxSide <= MO_MAX_MAX_DIM and lg <= MO_MAX_LONGEST_SIDE_PLUS_GIRTH:
            size = 'medium_oversized'
        elif weight <= LO_WEIGHT_LIMIT and maxSide <= LO_MAX_MAX_DIM and lg <= LO_MAX_LONGEST_SIDE_PLUS_GIRTH:
            size = 'large_oversized'
        elif weight >= SpO_WEIGHT_LIMIT or maxSide >= SpO_LONGEST_SIZE or lg >= SO_MAX_LONGEST_SIDE_PLUS_GIRTH:
            size = 'special_oversized'
        else:
            size = "N/A"
    except:
        size = "N/A"
    return size

def getDimension(page):
    """
    Scrapes the dimensions of the product in inches using regex.  Goes line by line checking if it's reached the
    product details yet.  Once it has, it starts using regex in the form _decimal_ x _decimal_ x _decimal_ [letter].
    Converts dimensions to inches if in centimeters or feet.

    :param page: Product page HMTL
    :return: [length, width, height] in inches
    """
    identifier = 'askSearchResultsHeader'
    #identifier = 'Product Dimensions'
    inDetails = False
    try:
        for line in page.splitlines():
            if identifier in line:
                inDetails = True
            if inDetails:
                try:
                    dimensions = re.search("(\d*\.?\d*) x (\d*\.?\d*) x (\d*\.?\d*) (\w)", line)
                    l = dimensions.group(1)
                    w = dimensions.group(2)
                    h = dimensions.group(3)
                    unit = dimensions.group(4)
                    break
                except:
                    pass
        if unit == 'c': # units in centimeters
            cm_to_inch = 2.54
            l = l * cm_to_inch
            w = w * cm_to_inch
            h = h * cm_to_inch
        elif unit == 'f': # units in feet
            ft_to_inch = 1/12
            l = l * ft_to_inch
            w = w * ft_to_inch
            h = h * ft_to_inch
        dim = [float(l), float(w), float(h)]
    except:
        dim = ['N/A', 'N/A', 'N/A']
    return dim

def getTitle(page):
    """
    Scrapes the product page for the name/title of the product listing

    :param page: Product page HTML
    :return: Title of the listing
    """
    try:
        i = 0
        for line in page.splitlines():
            i += 1
            if '<span id="productTitle"' in line:
                lineNum = i
        title = page.splitlines()[lineNum].lstrip()
    except:
        title = "N/A"
    return title

def getCategory(page):
    """
    Scrapes the product page for the product's broadest category using regex in the form: "in ____ (See top 100)".
    Replaces the "&amp" HTML format to just "&".

    :param page: Product page HTML
    :return: Broadest product category
    """
    try:
        category = re.search('in (.+?) \((.+?)>[S|s]ee [T|t]op 100', page).group(1)
        if "&amp;" in category:
            category = category.replace("&amp;", "&")
    except:
        category = "N/A"
    return category

def getReviews(page):
    """
    Scrapes the product page for the number of reviews on the product using regex.  Removes the commas from the
    number if greater than 1,000.

    :param page: Product page HTML
    :return: Number of reviews
    """
    try:
        reviews = re.search('<span id="acrCustomerReviewText" class="a-size-base">(.+?)</span>', page).group(1)
        reviews = re.sub("\D", "", reviews)
    except:
        reviews = "N/A"
    return reviews

def getBBP(page):
    """
    Scrapes the product listing page for the buy box price in dollars. Does not include the "$".
    Includes two diffrent product page formats.

    :param page: Product page HTML
    :return: Buy box price as float without "$"
    """
    try:
        BBP = re.search('<span id="priceblock_ourprice" class="a-size-medium a-color-price">(.+?)</span>', page).group(1)
        BBP = float(BBP[1:])
    except:
        try:
            BBP = re.search('<span class="buyingPrice">(.+?)</span>', page).group(1)
        except:
            BBP = "N/A"
    return BBP

def getBSR(page, cat):
    """
    Scrapes the product listing page for the best seller's rank using regex in the form "#__ in [category]".  Strips
    commas if the rank is greater than 1,000.

    :param page: Product page HTML
    :param cat: Broadest product category
    :return: Best sellers rank without commas
    """
    try:
        rank = re.search('#(.+?) in '+cat, page).group(1)
        # Strip reviews so only digits are left
        rank = re.sub("\D", "", rank)
    except:
        rank = "N/A"
    return rank

def getWeight(page):
    """
    Gets the product weight from the product listing page using regex on the line containing
    "View shipping rates and policies".  Just grabs the number on that line and then the first letter of the
    word next to it (the unit).  Converts the weight to pounds if in ounces or kilograms.

    :param page: Product page HTML
    :return: Product weight in pounds
    """
    identifier = "View shipping rates and policies"
    for line in page.splitlines():
        if identifier in line:
            # Search for the digit in the line
            weight = re.search(r'[\d\.\d]+', line).group()
            # get the index of that number
            index = line.index(weight)
            # get the first letter of the word after the digit
            unit = line[index + len(weight) + 1] # firstLetterOfUnit
            weight = float(weight)
            break
        else:
            weight = "N/A"
            unit = 'p'
    # Convert weight to pounds
    if unit == 'o': # ounces
        lb_in_oz = 1/16
        weight = weight * lb_in_oz
    elif unit == 'k': # kilograms
        lb_in_kg = 2.2
        weight = weight * lb_in_kg
    return weight

# -- Listings Page functions --

def getFBASellers(page):
    """
    Scrapes the listing page for the number of prime sellers selling new based on the number of occurances of the
    prime logo.

    :param page: Listings page HTML
    :return: Number of sellers selling new and FBA
    """
    # Assign the keyword to indicate a Prime Seller
    primeIdentifier = "aria-label=\"Amazon Prime TM\""
    # Count the number of time the keyword is in the string
    numPrimeSellers = counter(page,primeIdentifier)
    return numPrimeSellers

def isSoldByAmazon(page):
    """
    Scrapes the listing page for the number of Amazon logos found next to the Amazon.com seller name.  Returns
    True if it's found on the page.

    :param page: Listing page HTML
    :return: Boolean sold by Amazon
    """
    soldByAmazon = False
    identifier = "img alt=\"Amazon.com\""
    if counter(page, identifier) > 0:
        soldByAmazon = True
    return soldByAmazon

# Method to count the number of times a substring is found in a string
def counter(string, substring):
    """
    Counts the number of occurances of a substring in a string.  Used in the isSoldByAmazon() and getFBASellers()
    functions.

    :param string: string to search
    :param substring: string to search for
    :return: integer of number of occurances of the substring in the string
    """
    string_size = len(string)
    substring_size = len(substring)
    count = 0
    for i in range(0, string_size - substring_size + 1):
        if string[i:i + substring_size] == substring:
            count += 1
    return count

if __name__ == '__main__':
    main()
