## Hellenic API Documentation
This documentation covers the API endpoints for retrieving product and basic transaction interaction. The API utilizes Sanctum for authentication, requiring a valid API token for access.
### Base URLs
**Production**: api.hellenictravel.net<br>
**Sandbox**: develop.hellenictravel.network
### Authentication
All endpoints under the `/public-api/v1` prefix require authentication via Laravel Sanctum. Include a valid API token in the `Authorization` header as a Bearer token.
In order to obtain an API token, you need to have gone through the process of applying to become a partner, get accepted and then contact our team to get your corresponding API token for both Production and Sandbox.
### Headers
- All requests require the `Accept` header to be set to `application/json`, unless explicitly specified by a specific endpoint. 
- All responses default to English, for Spanish or Portuguese versions the `Content-Language` header must be manipulated, `es` for Spanish and `pt` for Portuguese.
### Products - Packages
These endpoints provide access to product packages.
#### 1. `GET /public-api/v1/products/packages`
- **Description:** Retrieves a list of available packages based on various filters.
- **Method:** `GET`
- **Query Parameters (Filters):**
    - `page_size` (optional): The number of packages to return per page. Must be an integer between 1 and 100. Defaults to 20. 
    - `date` (optional): The date for which to check package availability. Must be in `YYYY-MM-DD` format. If not provided, returns all packages with all categories and related data. The day of the week derived from this date is used to filter packages based on their availability. Dates before the current one are not allowed.
    - `min_days` (optional): Minimum duration (in days) of the package. Must be an integer between 0 and 30. Defaults to 0. 
    - `max_days` (optional): Maximum duration (in days) of the package. Must be an integer between 0 and 30. Defaults to 30. 
    - `detailed` (optional): Verbose response with more data for each package. Boolean (true/false), defaults to false.

- **Filter Usage:**
    - **`page_size`**: Controls the pagination size. For example, `?page_size=50` will return 50 packages per page.
    - **`date`**: Filters packages based on the day of the week. The API checks if the package is available on that particular day. For example, `?date=2025-07-20` (which is a Sunday) will only return packages available on Sundays. Ensure the date format is correct (`YYYY-MM-DD`) to avoid unexpected behavior. 
    - **`min_days` and `max_days`**: Filters packages based on their duration. For example, `?min_days=7&max_days=14` will return packages that last between 7 and 14 days.
    - **`detailed`**: Adds more data to each package regarding its description, included and not included commoditities.

- **Example Request:**
``` 
    GET /public-api/v1/products/packages?page_size=25&date=2025-07-21&min_days=3&max_days=10&detailed=true
```
##### This request retrieves packages, 25 per page, available on July 21st, 2025 (Monday), with a duration between 3 and 10 days, also providing extra information for each package.
- **Error Handling:** Returns a 404 error with code `000004` if no packages are found matching the criteria.
- **Notes:**
    - The API automatically filters packages based on the client's policy.
    - Packages are filtered to only include those with active status.
- **Response:**
    <br>**Base**__(`detailed=false`)__
    - `data`: Package data
        - `policy_commission`: (%) Commission based on the client making the request
        - `product_id`: Package UID
        - `title`: Package title
        - `days` and `nights`: Length of the package
        - `thumbnail_image`: Cover image of the package
        - `images`: (array) All images related to the package
            - `client_product_media_id`: ID of image
            - `type`: 1 is for images
            - `data`: URL of the image
        - `currency`: Data regarding each package designated currency.
            - `id`: ID of currency
            - `name`: Name of currency
            - `code`: Code of currency
            - `symbol`: Symbol of currency
        - `regions`: (array) List of regions which the package includes
            - `region`: Name of region
            - `days` and `nights`: Length of stay in specific region
            - `type_id` and `type`: Region type classification
                - `1`: Hotel region (overnight stay)
                - `2`: Airport/Flight region
                - `3`: Cruise region
                - `4`: Bus tour region
                - `5`: Bus tour with overnight stay
                - `6`: Cruise tour region
        - `stop_sales`(array): Set periods when the package is unavailable for booking
            - `id`: ID of entity
            - `start_date` and `end_date`: Affected period for restriction
            > If `date` is provided as a filter, then any package with an ongoing stop sale will not be fetched
    - `pagination`: Complete pagination data
        - `total`: Count of packages according to the filters/constraints
        - `total_pages`: Page count according to set `page_size`
        - `items_per_page`: Set or default `page_size`
        - `current_page`: Index of current page
        - `page_count`: Count of pages in the response(usually 1)
        - `previous_page`: URL for direct access to previous page, while keeping query parameters
        - `next_page`: URL for direct access to next page, while keeping query parameters
    
    **Verbose**__(`detailed=true`)__
    > In addition to **Base**
    - `description`: Detailed description of day-to-day program of the package
    - `included`: Package included benefits/commodities
    - `not_included`: Package excluded benefits/commodities
    - `categories`: (array) Available categories for the package
        - `category_type`: Name of category type
        - `category_id`: (**important**) Unique category ID of this specific category of the package, needed for creating transactions
        - `category_prices`: (array) Prices of category depending on the season
            - `category_price_id`: (**important**) Unique category price ID of this specific entry of the array, expressed by `season_from` and `season_to` (see below), needed for creating transactions
            - `season_from` and `season_to`: Period in which specific prices apply to the category
            - `(single/double/triple)_price\_(inc/excl)\_commission`: Detailed pricing for the 3 types of rooms with and without commission already calculated
            - `days_available`: (array) Dictates the availability of the category on a day-to-day basis (index 0 is Sunday and index 6 is Saturday)
    - `extras`: (array) Extras connected to the package(not always populated)
        - `extra_id`: Unique ID of given extra
        - `name`: Name of the extra
        - `required`: Boolean value indicating that the extra is not optional(compulsory based on the package)
        - `per_person`: Boolean value indicating if the extra applies the price per person or in the total
        - `price`: Price of the extra
        - `extra_type_id`: Extra type classification
            - `1`: Required extra (automatically included in transaction pricing)
            - `2`: Optional extra
            - `3`: Optional supplement extra
        > **Note:** When `required` is `true`, the extra price is automatically calculated and included in the total transaction cost during booking creation.

#### 2. `GET /public-api/v1/products/packages/{package_id}`
- **Description:** Retrieves a specific package by its UID (unique identifier).
- **Method:** `GET`
- **Parameters:**
    - (required): The UID of the package to retrieve. `package_id`

- **Query Parameters:**
    - `date` (optional): The date for which to check package availability. Must be in format. If not provided, returns all categories and related data. The day of the week derived from this date is used to filter package category prices based on their availability. `YYYY-MM-DD`

- **Filter Usage:**
    - **`date`**: Filters package category prices based on the day of the week. The API checks if the package category price is available on that particular day. For example, `?date=2025-07-22` will only return a package with category prices available on July 22nd, 2025. Ensure the date format is correct (`YYYY-MM-DD`) to avoid unexpected behavior. 

- **Example Request:**
``` 
    GET /public-api/v1/products/packages/{package_id}?date=2025-07-22
```
This request retrieves the package with the UID `package_id` and checks the package's category prices availability for July 22nd, 2025.
- **Error Handling:** 
    - Returns a 404 error with code `000004` if the package is not found.
    - Returns a 404 error with code `000007` if the package has an ongoing stop sale if given a date or if the date's day of the week is not available
- **Response:**
<br>
**Same as `detailed=true` from the previous request excluding pagination data since this is a singular object**

### Transactions
These endpoints manage transactions.
> For status IDs: 1 = Active, 2 = Paid, 3 = Cancelled, 4 = On Request, 5 = Expired, 6 = Cancellation Fee.
#### 1. `GET /public-api/v1/transactions`
- **Description:** Retrieves a list of transactions based on filters.
- **Method:** `GET`
- **Query Parameters (Filters):**
    - `page_size` (optional): The number of transactions to return per page. Defaults to 20 if not specified. Maximum value is 100, otherwise, it will be set to 20. 
    - `sub_client_locator` (optional): Filter by sub-client locator. 
    - `client_locator` (optional): Filter by client locator. 
    - `action_user_id` (optional): Filter by action user ID. 
    - `date_arrives` (optional): Filter by arrival date. 
    - `transaction_status_id` (optional): Filter by transaction status ID. 
    - `created_at` (optional): Filter by creation date (YYYY-MM-DD). 

- **Filter Usage:**
    - To use filters, append them as query parameters to the URL. For example, to filter transactions with a of `1`, use: `/public-api/v1/transactions?transaction_status_id=1`. `transaction_status_id`
    - The filter expects a date string in `YYYY-MM-DD` format. For example, `/public-api/v1/transactions?created_at=2025-07-17`. `created_at`
    - Multiple filters can be combined. For example, `/public-api/v1/transactions?transaction_status_id=1&date_arrives=2025-07-24`.
    - If a filter is not provided, all values for that field are included in the results.

- **Example Request:**
``` 
    GET /public-api/v1/transactions?page_size=10&transaction_status_id=1&date_arrives=2025-07-24
```
This request retrieves 10 transactions per page with a `transaction_status_id` of 1 and an arrival date of July 24th, 2025.
- **Error Handling:** Returns a 404 error with code `000004` if no transactions are found matching the criteria.

- **Response:**
    - `data`: Transaction data array
        - `transaction_id`: Transaction ID
        - `uid`: Transaction UID
        - `to_client_id`: ID of the receiving client
        - `to_client_name`: Name of the receiving client
        - `total_price`: Total price before commission
        - `final_total_price`: Final total price of the transaction
        - `paid_until`: Payment due date (YYYY-MM-DD HH:MM:SS)
        - `currency_id`: ID of currency
        - `currency_code`: Code of currency
        - `currency_symbol`: Symbol of currency
        - `transaction_status_id`: Status ID
        - `transaction_status_id_name`: Status name
        - `client_locator`: Client locator
        - `sub_client_locator`: Sub-client locator
        - `date_arrives`: Arrival date (YYYY-MM-DD HH:MM:SS)
        - `notes`: Notes (nullable)
        - `created_at`: Creation timestamp (ISO8601)
        - `transaction_rooms`: Room summary (object)
            - `total_customers`: Total number of customers
            - `total_single_rooms`: Total single rooms
            - `total_double_rooms`: Total double rooms
            - `total_triple_rooms`: Total triple rooms
        - `transaction_customers`: Array of customers
            - `transaction_customer_id`: Customer ID
            - `first_name`: First name
            - `last_name`: Last name
            - `gender`: Gender (1=Male, 2=Female)
    - `pagination`: Complete pagination data
        - `total`: Count of transactions according to the filters/constraints
        - `total_pages`: Page count according to set `page_size`
        - `items_per_page`: Set or default `page_size`
        - `current_page`: Index of current page
        - `page_count`: Count of pages in the response (usually 1)
        - `previous_page`: URL for direct access to previous page, while keeping query parameters
        - `next_page`: URL for direct access to next page, while keeping query parameters

#### 2. `GET /public-api/v1/transactions/{transaction_id}`
- **Description:** Retrieves a specific transaction by its ID.
- **Method:** `GET`
- **Parameters:**
    - (required): The ID of the transaction to retrieve. `transaction_id`

- **Example Request:**
``` 
    GET /public-api/v1/transactions/123
```
- **Error Handling:** Returns a 404 error with code `000004` if the transaction is not found

- **Response:**
    - `transaction_id`: Transaction ID
    - `uid`: Transaction UID
    - `to_client_id`: ID of the receiving client
    - `to_client_name`: Name of the receiving client
    - `total_price`: Total price before commission
    - `final_total_price`: Final total price of the transaction
    - `paid_until`: Payment due date (YYYY-MM-DD HH:MM:SS)
    - `currency_id`: ID of currency
    - `currency_code`: Code of currency
    - `currency_symbol`: Symbol of currency
    - `transaction_status_id`: Status ID
    - `transaction_status_id_name`: Status name
    - `client_locator`: Client locator
    - `sub_client_locator`: Sub-client locator
    - `date_arrives`: Arrival date (YYYY-MM-DD HH:MM:SS)
    - `notes`: Notes (nullable)
    - `created_at`: Creation timestamp (ISO8601)
    - `transaction_product`: Product information (object)
        - `transaction_product_id`: Transaction product ID
        - `quantity`: Quantity
        - `client_product_price`: Price information
            - `price_id`: Price ID
            - `product_category`: Category information
                - `category_id`: Category ID
                - `name`: Category name
                - `product`: Product details
                    - `client_product_id`: Product ID
                    - `title`: Product title
                    - `days`: Package duration in days
    - `transaction_rooms`: Room summary (object)
        - `total_customers`: Total number of customers
        - `total_single_rooms`: Total single rooms
        - `total_double_rooms`: Total double rooms
        - `total_triple_rooms`: Total triple rooms
    - `transaction_notes`: Array of transaction notes
    - `transaction_product_extras`: Array of product extras
    - `transaction_customers`: Array of customers
        - `transaction_customer_id`: Customer ID
        - `first_name`: First name
        - `last_name`: Last name
        - `gender`: Gender (1=Male, 2=Female)
    - `transaction_transportation`: Array of transportation segments
    - `transaction_documents`: Array of documents
        - `transaction_document_id`: Document ID
        - `document_url`: Document URL
        - `title`: Document title
        - `created_at`: Document creation timestamp
    - `transaction_accommodations`: Array of accommodations
        - `transaction_accommodation_id`: Accommodation ID
        - `check_in_date`: Check-in date/time
        - `check_out_date`: Check-out date/time
        - `client_country_region`: Region object
            - `name`: Region name
        - `pick_up_date`: Pick-up date/time
        - `name`: Accommodation name
        - `notes`: Accommodation notes
        - `confirmed_status_id`: Confirmation status ID
    - `transaction_rooms_customers`: Array of room-customer assignments
        - `room_type_id`: Room type ID
        - `room_customers`: Array of customers in the room
            - `transaction_customer_id`: Customer ID
            - `transaction_id`: Transaction ID
            - `gender_id`: Gender ID
            - `first_name`: First name
            - `last_name`: Last name
            - `age`: Age
            - `entity_status_id`: Entity status ID
            - `created_at`: Creation timestamp
            - `updated_at`: Update timestamp
            - `transaction_room_id`: Room ID

#### 3. `POST /public-api/v1/transactions/create`
- **Description:** Creates a new transaction.
- **Method:** `POST`
- **Request Body (JSON):**
    - `product_id` (required): The UID of the client product. 
    - `date_arrives` (required): The arrival date (YYYY-MM-DD). Must be after yesterday.
    - `category_id` (required): The ID of the desired category.
    - `category_price_id` (required): The ID of the desired price of the product based on the arrival date.  
    - `rooms` (required): An array of room configurations with customers.
        - `room_type_id` (required): The type of room (1=Single, 2=Double, 3=Triple)
        - `room_customers` (required): Array of customers for this room (must match room_type_id count)
            - `age` (required): Customer age (integer, minimum 0)
            - `first_name` (required): Customer first name (string, max 255 chars)
            - `gender_id` (required): Customer gender (1=Male, 2=Female)
            - `last_name` (required): Customer last name (string, max 255 chars)

- **Example Request:**
```json
POST /public-api/v1/transactions/create
Content-Type: application/json

{
    "product_id": 123,
    "category_id": 112,
    "category_price_id": 234,
    "date_arrives": "2025-07-25",
    "rooms": [
        {
            "room_type_id": 1,
            "room_customers": [
                {
                    "age": 55,
                    "first_name": "John",
                    "gender_id": 1,
                    "last_name": "Doe"
                }
            ]
        },
        {
            "room_type_id": 2,
            "room_customers": [
                {
                    "age": 35,
                    "first_name": "Jane",
                    "gender_id": 2,
                    "last_name": "Smith"
                },
                {
                    "age": 40,
                    "first_name": "Bob",
                    "gender_id": 1,
                    "last_name": "Johnson"
                }
            ]
        }
    ]
}
```

- **Error Handling:**
    - Returns a 404 error with code `000004` if the `product_id`, `category_id` or `category_price_id` are not found, or if the client doesn't have policy access to the product.
    - Returns a 406 error with code `000007` if the arrival date is outside the category price date range.
    - Returns a 406 error with code `000019` if the product is not available on the specified day of the week.
    - Returns a 409 error with code `000020` if there are stop sales for the product on the arrival date.
    - Returns a 422 error for validation failures (invalid data format, room customer count mismatch, etc.).

- **Response:**
A transaction object with the new data

#### 4. `DELETE /public-api/v1/transactions/{transaction_id}/cancel`
- **Description:** Cancels a specific transaction by its ID.
- **Method:** `DELETE`
- **Parameters:**
    - `transaction_id` (required): The UID of the transaction to cancel. 

- **Example Request:**
``` 
    DELETE /public-api/v1/transactions/123/cancel
```
- **Response:**
A transaction object with the new data (status changed to cancel and the total price equal to the cancellation policy equivalent)
> Payment for all services is required 30 days before the arrival of the clients or the commencement of the services to be provided by Hellenic Travel Network S.A. (HTN).
    45-30 days before arrival 25% cancellation fees;
    29-15 days before arrival 40% cancellation fees
    14-08 days before arrival 50% cancellation fees;
    07-01 days before arrival or No-Show 100% cancellation fees;

### General Notes
- Ensure that all dates are in the correct `YYYY-MM-DD` format.
- The API uses HTTP status codes to indicate the success or failure of a request.
- The API returns JSON responses.
