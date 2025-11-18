# smartling-configuration-editor.tsx

## File Metadata
- **Path**: `plugins/smartling/src/smartling-configuration-editor.tsx`
- **Type**: Text file
- **Size**: 4,822 bytes (154 lines)
- **Extension**: `.tsx`
- **Language**: TypeScript (React)

## Original Source
```tsx
/** @jsx jsx */
import { jsx } from '@emotion/core';
import { CustomReactEditorProps, fastClone } from './plugin-helpers';
import { observable, reaction, IReactionOptions, action } from 'mobx';
import React, { useEffect } from 'react';
import { useObserver, useLocalStore } from 'mobx-react';
import { Project, SmartlingApi } from './smartling';
import {
  CircularProgress,
  Select,
  MenuItem,
  ListItemText,
  Checkbox,
  Typography,
} from '@material-ui/core';

function useReaction<T = any>(
  expression: () => T,
  effect: (value: T) => void,
  options: IReactionOptions<any, any> = { fireImmediately: true }
): void {
  useEffect(() => reaction(expression, effect, options), []);
}

interface Props extends CustomReactEditorProps {
  api: SmartlingApi;
}

export const SmartlingConfigurationEditor: React.FC<Props> = props => {
  const store = useLocalStore(() => ({
    loading: false,
    projects: [] as any[],
    project: null as Project | null,
    filters: observable.map(props.value ? fastClone(props.value) : { project: '' }),
    targetLocales: [] as string[],
    catchError: (err: any) => {
      console.error('search error:', err);
      props.context.snackBar.show('There was an error searching for products');
      return null;
    },
    async fetchProject(projectId: string) {
      store.loading = true;
      this.project =
        (await props.api.getProject(projectId).catch(e => this.catchError(e)))?.project || null;
      // remove all target locales that are not in this project
      store.targetLocales = store.targetLocales.filter(locale =>
        this.project?.targetLocales.find(localeObj => localeObj.localeId === locale)
      );
      store.setValue();
      store.loading = false;
    },
    setValue() {
      props.onChange({
        ...fastClone(store.filters),
        targetLocales: store.targetLocales,
      });
    },
  }));

  useEffect(() => {
    store.targetLocales = props.value?.get('targetLocales') || [];
  }, [props.value]);

  useReaction(
    () => store.filters.get('project'),
    projectId => {
      if (projectId) {
        store.fetchProject(projectId);
      } else {
        store.project = null;
      }
    }
  );

  return useObserver(() => (
    <React.Fragment>
      <div css={{ marginBottom: 25, marginTop: 20 }}>
        <div
          css={{
            paddingLeft: 15,
            marginTop: 10,
            paddingBottom: 10,
            borderLeft: '1px solid #ccc',
          }}
        >
          {props.renderEditor({
            object: store.filters,
            fields: [
              {
                name: 'project',
                type: 'SmartlingProject',
                helperText: 'Project choice determines source locale',
                required: true,
              },
            ],
          })}
        </div>
      </div>
      <div css={{ marginBottom: 25, marginTop: 20 }}>
        <div
          css={{
            paddingLeft: 15,
            marginTop: 10,
            paddingBottom: 10,
            borderLeft: '1px solid #ccc',
          }}
        >
          <Typography>Target Locales*</Typography>
          {store.loading ? (
            <div css={{ textAlign: 'center' }}>
              <CircularProgress disableShrink size={20} />{' '}
            </div>
          ) : (
            <Select
              fullWidth
              value={store.targetLocales}
              placeholder="+ Add a value"
              renderValue={selected => (Array.isArray(selected) ? selected?.join(',') : selected)}
              onChange={action(event => {
                if (store.targetLocales.includes(event.target.value)) {
                  // remove
                  store.targetLocales = store.targetLocales.filter(
                    locale => locale !== event.target.value
                  );
                } else {
                  store.targetLocales = [event.target.value, ...store.targetLocales];
                }
                store.setValue();
              })}
            >
              {store.project ? (
                store.project.targetLocales.map(locale => (
                  <MenuItem key={locale.localeId} value={locale.localeId}>
                    <Checkbox
                      color="primary"
                      checked={store.targetLocales.includes(locale.localeId)}
                    />

                    <ListItemText primary={locale.description} />
                  </MenuItem>
                ))
              ) : (
                <Typography>Pick a project first</Typography>
              )}
            </Select>
          )}
          <Typography css={{ marginBottom: 15, marginTop: 10 }} variant="caption">
            Pick from the list of available target locales
          </Typography>
        </div>
      </div>
    </React.Fragment>
  ));
};

```


## High-Level Overview
remove all target locales that are not in this project

## Detailed Walkthrough

### Functions (1)

- `useReaction()`

### Imports (6)

- `mobx`
- `@emotion/core`
- `./plugin-helpers`
- `mobx-react`
- `react`
- `./smartling`

### Exports (1)

- `SmartlingConfigurationEditor`

## Usage Examples
*No usage examples found in source. Refer to tests or documentation.*

## Dependencies and Related Files
### Imports
- `mobx` (external)
- `@emotion/core` (external)
- `./plugin-helpers` (local)
- `mobx-react` (external)
- `react` (external)
- `@material-ui/core` (external)
- `./smartling` (local)

## Performance and Security Notes
*No specific performance or security concerns detected*

## Testing
*No test information available. Check for corresponding test files.*

---
*Generated by Repo Book Generator v1.0.0*
