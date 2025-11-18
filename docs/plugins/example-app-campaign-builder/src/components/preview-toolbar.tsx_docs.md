# preview-toolbar.tsx

## File Metadata
- **Path**: `plugins/example-app-campaign-builder/src/components/preview-toolbar.tsx`
- **Type**: Text file
- **Size**: 5,329 bytes (203 lines)
- **Extension**: `.tsx`
- **Language**: TypeScript (React)

## Original Source
```tsx
/** @jsx jsx */
import { jsx } from '@emotion/core'
import React from 'react'
import { useObserver, observer } from 'mobx-react'
import { ApplicationContext } from '../interfaces/application-context'
import { Tooltip, IconButton } from '@material-ui/core'
import {
  Undo,
  Fullscreen,
  LineStyle,
  Create,
  TabletMac,
  PhoneIphone,
  LaptopMac,
  Redo,
  Refresh,
} from '@material-ui/icons'

const context: ApplicationContext = require('@builder.io/app-context').default

interface ToolButtonsProps {
  tooltip?: string
  children: React.ReactNode
  active?: boolean
  onClick?: (event: React.MouseEvent<HTMLElement>) => void
  disabled?: boolean
  name?: string
}

const ToolButton = observer((props: ToolButtonsProps) => (
  <Tooltip title={props.tooltip}>
    <div>
      <IconButton
        onClick={props.onClick}
        disabled={props.disabled}
        css={{
          padding: 0,
          color: props.active ? '#670CB7' : '#999',
          width: 40,
          height: 38,
          opacity: props.disabled ? 0.5 : 1,
        }}
      >
        {props.children}
      </IconButton>
    </div>
  </Tooltip>
))

const resizeButtonStyles: any = {
  height: 38,
  width: 38,
  margin: '0 4px',
  position: 'relative',
  padding: 0,
}

export function PreviewToolbar() {
  return useObserver(() => {
    return (
      <div
        css={{
          display: 'flex',
          alignItems: 'center',
          backgroundColor: '#f8f8f8',
          minHeight: 39,
          position: 'relative',
        }}
      >
        <ToolButton
          tooltip="Full screen editing"
          active={context.contentEditorPage.fullScreenIframe}
          onClick={() => {
            context.contentEditorPage.fullScreenIframe =
              !context.contentEditorPage.fullScreenIframe
          }}
        >
          <Fullscreen />
        </ToolButton>

        <ToolButton
          name="content-edit-mode"
          tooltip="Content-only mode"
          active={context.contentEditorPage.contentEditingMode}
          onClick={() => {
            context.contentEditorPage.contentEditingMode =
              !context.contentEditorPage.contentEditingMode
          }}
        >
          <Create />
        </ToolButton>

        <ToolButton
          name="xray-mode"
          tooltip="X-Ray Mode"
          active={context.designerState.xrayMode}
          onClick={() => {
            context.designerState.xrayMode = !context.designerState.xrayMode
          }}
        >
          <LineStyle />
        </ToolButton>

        <div
          css={{
            display: 'flex',
            position: 'absolute',
            left: 0,
            right: 0,
            width: 200,
            margin: 'auto',
          }}
        >
          <IconButton
            css={resizeButtonStyles}
            onClick={() => {
              context.designerState.artboardSize.width = 2000
            }}
          >
            {/* TODO: highlight relevant color */}
            <LaptopMac css={{ color: '#c7c7c7' }} />
          </IconButton>
          <IconButton
            css={resizeButtonStyles}
            onClick={() => {
              context.designerState.artboardSize.width = 642
            }}
          >
            <TabletMac css={{ color: '#c7c7c7' }} />
          </IconButton>
          <IconButton
            css={resizeButtonStyles}
            onClick={() => {
              context.designerState.artboardSize.width = 321
            }}
          >
            <PhoneIphone css={{ color: '#c7c7c7' }} />
          </IconButton>
        </div>

        <div
          css={{
            display: 'flex',
            marginLeft: 'auto',
          }}
        >
          <Tooltip title="Undo">
            <div>
              <IconButton
                css={{ padding: 5 }}
                onClick={() => context.designerState.undo()}
                disabled={!context.designerState.canUndo}
              >
                <Undo
                  css={{
                    color: '#999',
                    opacity: context.designerState.canUndo ? 1 : 0.5,
                  }}
                />
              </IconButton>
            </div>
          </Tooltip>
          <Tooltip title="Redo">
            <div>
              <IconButton
                css={{ padding: 5 }}
                onClick={() => context.designerState.redo()}
                disabled={!context.designerState.canRedo}
              >
                <Redo
                  css={{
                    color: '#999',
                    opacity: context.designerState.canRedo ? 1 : 0.5,
                  }}
                />
              </IconButton>
            </div>
          </Tooltip>
          <Tooltip title="Refresh preview">
            <IconButton
              css={{
                opacity: 0.5,
                fontSize: 20,
                height: 40,
                width: 40,
                padding: 0,
              }}
              onClick={() => {
                const iframe = context.designerState.editingIframeRef
                if (iframe) {
                  // Reloads an iframe
                  iframe.src = iframe.src
                }
              }}
            >
              <Refresh />
            </IconButton>
          </Tooltip>
        </div>
      </div>
    )
  })
}

```


## High-Level Overview
JavaScript/TypeScript module. Contains 203 lines of code.

## Detailed Walkthrough

### Functions (1)

- `PreviewToolbar()`

### Imports (5)

- `@emotion/core`
- `mobx-react`
- `../interfaces/application-context`
- `react`
- `@material-ui/core`

### Exports (1)

- `PreviewToolbar`

## Usage Examples
*No usage examples found in source. Refer to tests or documentation.*

## Dependencies and Related Files
### Imports
- `@material-ui/icons` (external)
- `@emotion/core` (external)
- `mobx-react` (external)
- `../interfaces/application-context` (local)
- `react` (external)
- `@material-ui/core` (external)

## Performance and Security Notes
*No specific performance or security concerns detected*

## Testing
*No test information available. Check for corresponding test files.*

---
*Generated by Repo Book Generator v1.0.0*
