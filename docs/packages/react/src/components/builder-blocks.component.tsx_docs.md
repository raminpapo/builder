# builder-blocks.component.tsx

## File Metadata
- **Path**: `packages/react/src/components/builder-blocks.component.tsx`
- **Type**: Text file
- **Size**: 4,936 bytes (178 lines)
- **Extension**: `.tsx`
- **Language**: TypeScript (React)

## Original Source
```tsx
/** @jsx jsx */
import { jsx } from '@emotion/core';
import React from 'react';
import { Builder } from '@builder.io/sdk';
import { BuilderBlock } from './builder-block.component';
// TODO: fetch these for user and send them with same response like graphql
import { Size } from '../constants/device-sizes.constant';
import ReactDOM from 'react-dom';
import { BuilderStoreContext } from '../store/builder-store';

export interface BuilderBlocksProps {
  fieldName?: string;
  // TODO: block type (ElementType[])
  blocks?: any[] | React.ReactNode;
  child?: boolean;
  // TODO: use new react context for this to pass window size all the way down
  size?: Size;
  style?: React.CSSProperties;
  parentElementId?: string;
  parent?: any;
  dataPath?: string;
  className?: string;
  emailMode?: boolean;
}

interface BuilderBlocksState {
  // width: number
}

// TODO: options to set direciotn
export class BuilderBlocks extends React.Component<BuilderBlocksProps, BuilderBlocksState> {
  get isRoot() {
    return !this.props.child;
  }

  get noBlocks() {
    const { blocks } = this.props;
    return !(blocks && (blocks as any).length); // TODO: allow react nodes
  }
  get path() {
    const pathPrefix = 'component.options.';
    let path = this.props.dataPath || '';
    const thisPrefix = 'this.';
    if (path.trim()) {
      if (path.startsWith(thisPrefix)) {
        path = path.replace(thisPrefix, '');
      } else if (!path.startsWith(pathPrefix)) {
        path = pathPrefix + path;
      }
    }
    return path;
  }

  get parentId() {
    if (this.props.parentElementId) {
      return this.props.parentElementId;
    }
    return this.props.parent && this.props.parent.id;
  }

  onClickEmptyBlocks = () => {
    if (Builder.isIframe && this.noBlocks) {
      window.parent?.postMessage(
        {
          type: 'builder.clickEmptyBlocks',
          data: {
            parentElementId: this.parentId,
            dataPath: this.path,
          },
        },
        '*'
      );
    }
  };
  onHoverEmptyBlocks = () => {
    if (Builder.isEditing && this.noBlocks) {
      window.parent?.postMessage(
        {
          type: 'builder.hoverEmptyBlocks',
          data: {
            parentElementId: this.parentId,
            dataPath: this.path,
          },
        },
        '*'
      );
    }
  };

  // <!-- Builder Blocks --> in comments hmm
  render() {
    const { blocks } = this.props;

    const TagName = this.props.emailMode ? 'span' : 'div';

    // TODO: how deep check this automatically for mobx... hmmm optional / peer dependency?
    return (
      // TODO: component <Stack direction="vertical">
      // TODO: react.fragment instead?
      <TagName
        className={
          'builder-blocks' +
          (this.noBlocks ? ' no-blocks' : '') +
          (this.props.child ? ' builder-blocks-child' : '') +
          (this.props.className ? ' ' + this.props.className : '')
        }
        builder-type="blocks"
        // TODO: only fi in iframe?
        builder-path={Builder.isIframe ? this.path : undefined}
        builder-parent-id={this.parentId}
        css={
          {
            ...(!this.props.emailMode && {
              display: 'flex',
              flexDirection: 'column',
              alignItems: 'stretch',
            }),
            ...this.props.style,
          } as any
        }
        onClick={() => {
          if (this.noBlocks) {
            this.onClickEmptyBlocks();
          }
        }}
        {...(Builder.isEditing && {
          onMouseEnter: () => this.onHoverEmptyBlocks(),
        })}
      >
        {/* TODO: if is react node (for react compatibility) render it */}
        {/* TODO: maybe don't do this to preserve blocks always editable */}
        {(blocks &&
          Array.isArray(blocks) &&
          (blocks as any[]).map((block, index) =>
            block && block['@type'] === '@builder.io/sdk:Element' ? (
              <BuilderBlock
                key={block.id}
                block={block}
                index={index}
                fieldName={this.props.fieldName}
                child={this.props.child}
                emailMode={this.props.emailMode}
              />
            ) : (
              block
            )
          )) ||
          blocks}
      </TagName>
    );
  }

  static renderInto(
    elementOrSelector: string | HTMLElement,
    props: BuilderBlocksProps = {},
    builderState: any
  ) {
    if (!elementOrSelector) {
      return;
    }

    let element: Element | null = null;

    if (typeof elementOrSelector === 'string') {
      element = document.querySelector(elementOrSelector);
    } else {
      if (elementOrSelector instanceof Element) {
        element = elementOrSelector;
      }
    }
    return ReactDOM.render(
      <BuilderStoreContext.Provider value={builderState}>
        <BuilderBlocks {...props} />
      </BuilderStoreContext.Provider>,
      element
    );
  }
}

```


## High-Level Overview
TODO: fetch these for user and send them with same response like graphql
TODO: block type (ElementType[])
TODO: use new react context for this to pass window size all the way down
width: number
TODO: options to set direciotn

## Detailed Walkthrough


### Imports (7)

- `@builder.io/sdk`
- `../constants/device-sizes.constant`
- `react-dom`
- `@emotion/core`
- `./builder-block.component`
- `../store/builder-store`
- `react`

### Exports (2)

- `interface`
- `BuilderBlocks`

## Usage Examples
*No usage examples found in source. Refer to tests or documentation.*

## Dependencies and Related Files
### Imports
- `@builder.io/sdk` (external)
- `../constants/device-sizes.constant` (local)
- `react-dom` (external)
- `@emotion/core` (external)
- `./builder-block.component` (local)
- `../store/builder-store` (local)
- `react` (external)

## Performance and Security Notes
*No specific performance or security concerns detected*

## Testing
*No test information available. Check for corresponding test files.*

---
*Generated by Repo Book Generator v1.0.0*
