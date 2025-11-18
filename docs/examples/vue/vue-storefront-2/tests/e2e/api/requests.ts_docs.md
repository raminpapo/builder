# requests.ts

## File Metadata
- **Path**: `examples/vue/vue-storefront-2/tests/e2e/api/requests.ts`
- **Type**: Text file
- **Size**: 4,953 bytes (242 lines)
- **Extension**: `.ts`
- **Language**: TypeScript

## Original Source
```typescript
import { Address, Customer, Product } from '../types/types';

export type CreateCartResponse = {
  body: {
    data: {
      cart: {
        id: string;
      };
    };
  };
};

export type CreateMyOrderFromCartResponse = {
  body: {
    data: {
      order: {
        id: string;
      };
    };
  };
};

export type CustomerSignMeInResponse = {
  body: {
    data: {
      user: {
        customer: {
          firstName: string;
          lastName: string;
          email: string;
        };
      };
    };
  };
};

export type GetMeResponse = {
  body: {
    data: {
      me: {
        customer: {
          firstName: string;
          lastName: string;
          email: string;
        };
      };
    };
  };
};

export type GetShippingMethodsResponse = {
  body: {
    data: {
      shippingMethods: [
        {
          id: string;
          name: string;
        }
      ];
    };
  };
};

const requests = {
  addToCart(cartId: string, product: Product, quantity?: number): Cypress.Chainable {
    const options = {
      url: '/api/ct/addToCart',
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
      body: [
        {
          id: cartId,
          version: 1,
        },
        {
          product: {
            id: product.id,
            sku: product.sku,
          },
          quantity: quantity ?? 1,
        },
        null,
      ],
    };
    return cy.request(options);
  },

  createCart(): Cypress.Chainable {
    const options = {
      url: '/api/ct/createCart',
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
      body: [{}, null],
    };
    return cy.request(options);
  },

  createMyOrderFromCart(id, version?): Cypress.Chainable {
    const options = {
      url: '/api/ct/createMyOrderFromCart',
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
      body: [
        {
          id: id,
          version: version ?? 1,
        },
        null,
      ],
    };
    return cy.request(options);
  },

  customerSignMeIn(customer: Customer): Cypress.Chainable {
    const options = {
      url: '/api/ct/customerSignMeIn',
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
      body: [
        {
          email: customer.email,
          password: customer.password,
        },
      ],
    };
    return cy.request(options);
  },

  customerSignMeUp(customer: Customer): Cypress.Chainable {
    const options = {
      url: '/api/ct/customerSignMeUp',
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
      body: [
        {
          email: customer.email,
          password: customer.password,
          firstName: customer.firstName,
          lastName: customer.lastName,
        },
      ],
    };
    return cy.request(options);
  },

  getMe(customer?: boolean): Cypress.Chainable {
    const options = {
      url: '/api/ct/getMe',
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
      body: [{ customer: customer ?? false }, null],
    };
    return cy.request(options);
  },

  getShippingMethods(cartId: string): Cypress.Chainable {
    const options = {
      url: '/api/ct/getShippingMethods',
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
      body: [cartId],
    };
    return cy.request(options);
  },

  updateCart(
    cartId: string,
    data?: { addresses?: { shipping?: Address; billing?: Address }; shippingMethodId?: string }
  ): Cypress.Chainable {
    const actions = [];

    if (data.addresses !== undefined) {
      if (data.addresses.shipping !== undefined)
        actions.push({
          setShippingAddress: {
            address: {
              ...data.addresses.shipping,
            },
          },
        });

      if (data.addresses.billing !== undefined)
        actions.push({
          setBillingAddress: {
            address: {
              ...data.addresses.billing,
            },
          },
        });
    }

    if (data.shippingMethodId !== undefined)
      actions.push({
        setShippingMethod: {
          shippingMethod: {
            id: data.shippingMethodId,
          },
        },
      });

    const options = {
      url: '/api/ct/updateCart',
      method: 'POST',
      headers: {
        Accept: 'application/json',
        'Content-Type': 'application/json',
      },
      body: [
        {
          id: cartId,
          version: 1,
          actions: [...actions],
        },
        null,
      ],
    };
    return cy.request(options);
  },
};

export default requests;

```


## High-Level Overview
JavaScript/TypeScript module. Contains 242 lines of code.

## Detailed Walkthrough


### Imports (1)

- `../types/types`

### Exports (6)

- `type`
- `requests`

## Usage Examples
*No usage examples found in source. Refer to tests or documentation.*

## Dependencies and Related Files
### Imports
- `../types/types` (local)

## Performance and Security Notes
🔒 **Security**: Contains password/secret references - verify proper handling

## Testing
*No test information available. Check for corresponding test files.*

---
*Generated by Repo Book Generator v1.0.0*
