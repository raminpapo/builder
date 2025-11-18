# e2e-checkout-shipping-validation.json

## File Metadata
- **Path**: `examples/vue/vue-storefront-2/tests/e2e/fixtures/test-data/e2e-checkout-shipping-validation.json`
- **Type**: Text file
- **Size**: 6,606 bytes (259 lines)
- **Extension**: `.json`
- **Language**: JSON

## Original Source
```json
{
  "Should successfully save address - guest customer": {
    "product": {
      "name": "Shirt Aspesi white M",
      "id": 1,
      "sku": "M0E20000000ED0W",
      "size": 34
    },
    "customer": {
      "address": {
        "shipping": {
          "firstName": "Jane",
          "lastName": "Doe",
          "streetName": "Vuestorefront Rd.",
          "streetNumber": "13/37",
          "city": "Los Angeles",
          "state": "California",
          "country": "United States",
          "postalCode": "90001",
          "phone": "+12345678910"
        }
      }
    },
    "shippingMethod": "Express US"
  },
  "Should successfully save address - registered customer": {
    "product": {
      "name": "Shirt Aspesi white M",
      "id": 1,
      "sku": "M0E20000000ED0W",
      "size": 34
    },
    "customer": {
      "firstName": "Jane",
      "lastName": "Doe",
      "password": "vuestorefront",
      "address": {
        "shipping": {
          "firstName": "Jane",
          "lastName": "Doe",
          "streetName": "Vuestorefront Rd.",
          "streetNumber": "13/37",
          "city": "Los Angeles",
          "state": "California",
          "country": "United States",
          "postalCode": "90001",
          "phone": "+12345678910"
        }
      }
    },
    "shippingMethod": "Express US"
  },
  "Should display an error - First Name empty": {
    "product": {
      "name": "Shirt Aspesi white M",
      "id": 1,
      "sku": "M0E20000000ED0W",
      "size": 34
    },
    "customer": {
      "address": {
        "shipping": {
          "lastName": "Doe",
          "streetName": "Vuestorefront Rd.",
          "streetNumber": "13/37",
          "city": "Los Angeles",
          "state": "California",
          "country": "United States",
          "postalCode": "90001",
          "phone": "+12345678910"
        }
      }
    },
    "errorMessage": "Please fill out this field."  
  },
  "Should display an error - Last Name empty": {
    "product": {
      "name": "Shirt Aspesi white M",
      "id": 1,
      "sku": "M0E20000000ED0W",
      "size": 34
    },
    "customer": {
      "address": {
        "shipping": {
          "firstName": "Jane",
          "streetName": "Vuestorefront Rd.",
          "streetNumber": "13/37",
          "city": "Los Angeles",
          "state": "California",
          "country": "United States",
          "postalCode": "90001",
          "phone": "+12345678910"
        }
      }
    },
    "errorMessage": "Please fill out this field."  
  },
  "Should display an error - Street Name empty": {
    "product": {
      "name": "Shirt Aspesi white M",
      "id": 1,
      "sku": "M0E20000000ED0W",
      "size": 34
    },
    "customer": {
      "address": {
        "shipping": {
          "firstName": "Jane",
          "lastName": "Doe",
          "streetNumber": "13/37",
          "city": "Los Angeles",
          "state": "California",
          "country": "United States",
          "postalCode": "90001",
          "phone": "+12345678910"
        }
      }
    },
    "errorMessage": "Please fill out this field." 
  },
  "Should display an error - Street Number empty": {
    "product": {
      "name": "Shirt Aspesi white M",
      "id": 1,
      "sku": "M0E20000000ED0W",
      "size": 34
    },
    "customer": {
      "address": {
        "shipping": {
          "firstName": "Jane",
          "lastName": "Doe",
          "streetName": "Vuestorefront Rd.",
          "city": "Los Angeles",
          "state": "California",
          "country": "United States",
          "postalCode": "90001",
          "phone": "+12345678910"
        }
      }
    },
    "errorMessage": "Please fill out this field." 
  },
  "Should display an error - City empty": {
    "product": {
      "name": "Shirt Aspesi white M",
      "id": 1,
      "sku": "M0E20000000ED0W",
      "size": 34
    },
    "customer": {
      "address": {
        "shipping": {
          "firstName": "Jane",
          "lastName": "Doe",
          "streetName": "Vuestorefront Rd.",
          "streetNumber": "13/37",
          "state": "California",
          "country": "United States",
          "postalCode": "90001",
          "phone": "+12345678910"
        }
      }
    },
    "errorMessage": "Please fill out this field." 
  },
  "Should display an error - Postal Code empty": {
    "product": {
      "name": "Shirt Aspesi white M",
      "id": 1,
      "sku": "M0E20000000ED0W",
      "size": 34
    },
    "customer": {
      "address": {
        "shipping": {
          "firstName": "Jane",
          "lastName": "Doe",
          "streetName": "Vuestorefront Rd.",
          "streetNumber": "13/37",
          "city": "Los Angeles",
          "state": "California",
          "country": "United States",
          "phone": "+12345678910"
        }
      }
    },
    "errorMessage": "Please fill out this field." 
  },
  "Should display an error - Phone empty": {
    "product": {
      "name": "Shirt Aspesi white M",
      "id": 1,
      "sku": "M0E20000000ED0W",
      "size": 34
    },
    "customer": {
      "address": {
        "shipping": {
          "firstName": "Jane",
          "lastName": "Doe",
          "streetName": "Vuestorefront Rd.",
          "streetNumber": "13/37",
          "city": "Los Angeles",
          "state": "California",
          "country": "United States",
          "postalCode": "90001"
        }
      }
    },
    "errorMessage": "Please fill out this field." 
  },
  "Should display an error - Country empty": {
    "product": {
      "name": "Shirt Aspesi white M",
      "id": 1,
      "sku": "M0E20000000ED0W",
      "size": 34
    },
    "customer": {
      "address": {
        "shipping": {
          "firstName": "Jane",
          "lastName": "Doe",
          "streetName": "Vuestorefront Rd.",
          "streetNumber": "13/37",
          "city": "Los Angeles",
          "postalCode": "90001",
          "phone": "+12345678910"
        }
      }
    },
    "errorMessage": "This field is required" 
  },
  "Should display an error - State empty": {
    "product": {
      "name": "Shirt Aspesi white M",
      "id": 1,
      "sku": "M0E20000000ED0W",
      "size": 34
    },
    "customer": {
      "address": {
        "shipping": {
          "firstName": "Jane",
          "lastName": "Doe",
          "streetName": "Vuestorefront Rd.",
          "streetNumber": "13/37",
          "city": "Los Angeles",
          "country": "United States",
          "postalCode": "90001",
          "phone": "+12345678910"
        }
      }
    },
    "errorMessage": "This field is required" 
  }
}

```


## High-Level Overview
JSON configuration or data file.

## Detailed Walkthrough

*No structured elements detected*

## Usage Examples
*No usage examples found in source. Refer to tests or documentation.*

## Dependencies and Related Files
*No explicit dependencies found*

## Performance and Security Notes
🔒 **Security**: Contains password/secret references - verify proper handling

## Testing
*No test information available. Check for corresponding test files.*

---
*Generated by Repo Book Generator v1.0.0*
