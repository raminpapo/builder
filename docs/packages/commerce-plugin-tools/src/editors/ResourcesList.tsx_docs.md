# ResourcesList.tsx

## File Metadata
- **Path**: `packages/commerce-plugin-tools/src/editors/ResourcesList.tsx`
- **Type**: Text file
- **Size**: 5,569 bytes (192 lines)
- **Extension**: `.tsx`
- **Language**: TypeScript (React)

## Original Source
```tsx
/** @jsx jsx */
import { jsx } from '@emotion/core';
import React, { useEffect } from 'react';
import { action } from 'mobx';
import { useObserver, useLocalStore } from 'mobx-react';
import {
  ResourcePreviewCell,
  ResourcePreviewCellProps,
  ResourcePicker,
  ResourcePickerProps,
} from './ResourcesPicker';
import { CircularProgress, Button, Typography, Tooltip, IconButton } from '@material-ui/core';
import { Close } from '@material-ui/icons';
import appState from '@builder.io/app-context';
import { Resource } from '../interfaces/resource';
import capitalize from 'lodash/capitalize';
import pluralize from 'pluralize';
import { CommerceAPIOperations } from '..';

export type PickResourceListProps = {
  api: CommerceAPIOperations;
  value?: string[];
  onChange(newValue: string[]): void;
  resourcePicker?: React.FC<ResourcePickerProps>;
  onDone(): void;
  resourceName: string;
};

const ResourcePreviewById = (
  props: {
    id: string;
    api: CommerceAPIOperations;
    resourceName: string;
  } & Partial<ResourcePreviewCellProps>
) => {
  const { id, ...rest } = props;
  const store = useLocalStore(() => ({
    loading: false,
    resourceInfo: null as Resource | null,
    async getResource() {
      this.loading = true;
      try {
        const value = await props.api[props.resourceName].findById(props.id);
        this.resourceInfo = value;
      } catch (e) {
        console.error(e);
      }
      this.loading = false;
    },
  }));
  useEffect(() => {
    store.getResource();
  }, []);

  return useObserver(() => {
    // TODO: HTTP cache, while loading show placeholder loading

    if (store.loading) {
      // TODO: fancy material placeholders
      return <CircularProgress disableShrink size={20} />;
    }
    return (
      (store.resourceInfo && <ResourcePreviewCell resource={store.resourceInfo} {...rest} />) || (
        <React.Fragment></React.Fragment>
      )
    );
  });
};

export function PickResourceList(props: PickResourceListProps) {
  const store = useLocalStore(() => ({
    get value() {
      return props.value || [];
    },
  }));
  return useObserver(() => {
    return (
      <React.Fragment>
        <Typography variant="caption" css={{ paddingBottom: 15, textAlign: 'center' }}>
          Choose {pluralize.plural(props.resourceName)}
        </Typography>
        <div>
          {store.value?.map((item, index) => (
            <div
              css={{
                display: 'flex',
                '&:hover button': {
                  opacity: 1,
                },
              }}
              key={index}
            >
              <ResourcePreviewById
                resourceName={props.resourceName}
                key={item}
                id={item}
                api={props.api}
              />
              <Tooltip title="Remove product">
                <IconButton
                  css={{
                    opacity: 0,
                    transition: 'opacity 0.2s ease-in-out',
                    padding: 5,
                    marginLeft: 'auto',
                    alignSelf: 'center',
                  }}
                  onClick={() => {
                    const res = [
                      ...props.value!.slice(0, props.value!.indexOf(item)),
                      ...props.value!.slice(props.value!.indexOf(item) + 1),
                    ];
                    props.onChange(res);
                  }}
                >
                  <Close />
                </IconButton>
              </Tooltip>
            </div>
          ))}
        </div>
        {/* On click - choose product */}
        <Button
          color="primary"
          variant="outlined"
          fullWidth
          css={{ marginTop: 10 }}
          onClick={() => {
            const { value, resourcePicker, ...rest } = props;
            const PickerCompnent = resourcePicker || ResourcePicker;
            const close = appState.globalState.openDialog(
              <PickerCompnent
                {...rest}
                context={appState}
                omitIds={store.value}
                onChange={action(resource => {
                  if (resource) {
                    props.onChange([...(store.value || []), String(resource.id)]);
                  }
                  close();
                })}
              />
            );
          }}
        >
          + {capitalize(props.resourceName)}
        </Button>
        <Button
          color="primary"
          variant="outlined"
          fullWidth
          css={{ marginTop: 10 }}
          onClick={props.onDone}
        >
          Done
        </Button>
      </React.Fragment>
    );
  });
}

export function PickResourcesListButton(props: Omit<PickResourceListProps, 'onDone'>) {
  useEffect(() => {
    if (typeof props.value === 'undefined') {
      try {
        props.onChange([]);
      } catch (e) {
        console.warn('coudnt set default value', e);
      }
    }
  }, []);

  return useObserver(() => {
    return (
      <React.Fragment>
        <Button
          onClick={() => {
            const close = appState.globalState.openDialog(
              <div css={{ padding: 30, width: 500, maxWidth: '90vw' }}>
                <PickResourceList {...props} onDone={() => close()} />
              </div>
            );
          }}
          color="inherit"
          css={{ color: '#999', whiteSpace: 'nowrap' }}
        >
          {props.value?.length || 0} {pluralize(props.resourceName, props.value?.length || 0)}
        </Button>
      </React.Fragment>
    );
  });
}

```


## High-Level Overview
JavaScript/TypeScript module. Contains 192 lines of code.

## Detailed Walkthrough

### Functions (3)

- `PickResourceList()`
- `PickResourcesListButton()`
- `ResourcePreviewById()`

### Imports (11)

- `@material-ui/icons`
- `@builder.io/app-context`
- `pluralize`
- `mobx`
- `../interfaces/resource`
- `@emotion/core`
- `mobx-react`
- `..`
- `react`
- `@material-ui/core`
- `lodash/capitalize`

### Exports (3)

- `type`
- `PickResourcesListButton`
- `PickResourceList`

## Usage Examples
*No usage examples found in source. Refer to tests or documentation.*

## Dependencies and Related Files
### Imports
- `@material-ui/icons` (external)
- `@builder.io/app-context` (external)
- `pluralize` (external)
- `mobx` (external)
- `../interfaces/resource` (local)
- `@emotion/core` (external)
- `./ResourcesPicker` (local)
- `mobx-react` (external)
- `..` (local)
- `react` (external)
- `@material-ui/core` (external)
- `lodash/capitalize` (external)

## Performance and Security Notes
*No specific performance or security concerns detected*

## Testing
*No test information available. Check for corresponding test files.*

---
*Generated by Repo Book Generator v1.0.0*
