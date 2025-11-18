# service.ts

## File Metadata
- **Path**: `plugins/magento2/src/service.ts`
- **Type**: Text file
- **Size**: 4,357 bytes (160 lines)
- **Extension**: `.ts`
- **Language**: TypeScript

## Original Source
```typescript
import pkg from '../package.json';
const productSearchGql = (search: string, urlKey?: string) => `
{
  products(search: "${search || ''}"  ${urlKey ? `, filter: { url_key: { eq: "${urlKey}"}}` : ''}) {
    items {
      name
      url_key
      uid
      image {
        url
      }
    }
  }
}
`;

// TODO: search is not possible, figure out workarounds
const categoriesSearchGql = (_search: string, url_key?: string) => `
{
  categories ${url_key ? `(filters: { url_key: { eq: "${url_key}"}})` : ''} {
    items {
      image
      name
      uid
      url_key
    }
  }
}
`;

const proxyUrl = (endUrl: string) => {
  return `https://builder.io/api/v1/proxy-api?url=${encodeURIComponent(endUrl)}`;
};

export const getAPI = (storeUrl: string) => {
  const transformProduct = (product: any) => ({
    ...product,
    image: {
      src: product.image.url,
    },
    title: product.name,
    handle: product.url_key,
    id: product.url_key,
  });

  const transformCategory = (category: any) => ({
    ...category,
    title: category.name,
    handle: category.url_key,
    image: {
      src: category.image,
    },
    id: category.url_key,
  });

  const endpoint = proxyUrl(`${storeUrl}/graphql`);
  const headers = {
    'content-type': 'application/json',
  };

  // No clear way to get a product by ID, opted to make urlKey both the id and the handle
  return {
    product: {
      async findById(id: string) {
        const query = productSearchGql('', id);
        const result = await fetch(endpoint, {
          method: 'POST',
          headers,
          body: JSON.stringify({
            query,
          }),
        }).then(res => res.json());
        return transformProduct(result.data.products.items[0]);
      },
      async findByHandle(handle: string) {
        const query = productSearchGql('', handle);
        const result = await fetch(endpoint, {
          method: 'POST',
          headers,
          body: JSON.stringify({
            query,
          }),
        }).then(res => res.json());
        return transformProduct(result.data.products.items[0]);
      },
      async search(search: string) {
        const query = productSearchGql(search);
        const result = await fetch(endpoint, {
          method: 'POST',
          headers,
          body: JSON.stringify({
            query,
          }),
        }).then(res => res.json());
        return result.data.products.items.map((item: any) => transformProduct(item));
      },
      getRequestObject(id: string) {
        return {
          '@type': '@builder.io/core:Request' as const,
          request: {
            // TODO: figure out how to get a public url of the product, might not be possible
            url: ``,
          },
          options: {
            product: id,
            pluginId: pkg.name,
          },
        };
      },
    },
    category: {
      async findById(id: string) {
        const query = categoriesSearchGql('', id);
        const result = await fetch(endpoint, {
          method: 'POST',
          headers,
          body: JSON.stringify({
            query,
          }),
        }).then(res => res.json());
        return transformCategory(result.data.categories.items[0]);
      },
      async findByHandle(handle: string) {
        const query = categoriesSearchGql('', handle);
        const result = await fetch(endpoint, {
          method: 'POST',
          headers,
          body: JSON.stringify({
            query,
          }),
        }).then(res => res.json());
        return transformCategory(result.data.categories.items[0]);
      },
      async search(search: string) {
        const query = categoriesSearchGql(search);
        const result = await fetch(endpoint, {
          method: 'POST',
          headers,
          body: JSON.stringify({
            query,
          }),
        }).then(res => res.json());
        return result.data.categories.items.map((item: any) => transformCategory(item));
      },
      getRequestObject(id: string) {
        return {
          '@type': '@builder.io/core:Request' as const,
          request: {
            // TODO: figure out how to get a public url of the category, might not be possible
            url: ``,
          },
          options: {
            category: id,
            pluginId: pkg.name,
          },
        };
      },
    },
  };
};

```


## High-Level Overview
TODO: search is not possible, figure out workarounds

## Detailed Walkthrough

### Functions (6)

- `productSearchGql()`
- `categoriesSearchGql()`
- `proxyUrl()`
- `getAPI()`
- `transformProduct()`
- `transformCategory()`

### Imports (1)

- `../package.json`

### Exports (1)

- `getAPI`

## Usage Examples
*No usage examples found in source. Refer to tests or documentation.*

## Dependencies and Related Files
### Imports
- `../package.json` (local)

## Performance and Security Notes
*No specific performance or security concerns detected*

## Testing
*No test information available. Check for corresponding test files.*

---
*Generated by Repo Book Generator v1.0.0*
