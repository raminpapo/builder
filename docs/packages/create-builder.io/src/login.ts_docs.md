# login.ts

## File Metadata
- **Path**: `packages/create-builder.io/src/login.ts`
- **Type**: Text file
- **Size**: 4,251 bytes (165 lines)
- **Extension**: `.ts`
- **Language**: TypeScript

## Original Source
```typescript
import { join } from 'path';
import { userInfo } from 'os';
import { readJSON, existsSync, outputJSON } from 'fs-extra';
import http from 'http';
import { bold, yellowBright } from 'colorette';
import { Spinner } from 'cli-spinner';
import { HOST, openBuilderAuth } from './open';
import { askQuestion } from './utils';
import { cliOptions } from './cli';

interface Credentials {
  version: '1';
  privateKey?: string;
  apiKey?: string;
}

let credential: Credentials | undefined;

export interface LoginOpts {
  redirectURL?: string;
}

export const getCredentialsFilePath = () => {
  const info = userInfo();
  return join(info.homedir, '.config', 'builder', 'credentials.json');
};

export const getLogin = async (opts?: LoginOpts) => {
  if (credential) {
    return credential;
  }
  const credentialsPath = getCredentialsFilePath();
  try {
    if (existsSync(credentialsPath)) {
      const file = await readJSON(credentialsPath);
      if (typeof file === 'object' && file != null && file.version === '1') {
        credential = file;
        return credential;
      }
    }
  } catch (err) {
    console.error(err);
    return undefined;
  }
  return await login(opts);
};

export const mustLogin = async (opts?: LoginOpts) => {
  const login = await getLogin(opts);
  if (!login) {
    throw new Error('Login with builder.io failed');
  }
  return login;
};

export const mustGetApiKey = async (opts?: LoginOpts) => {
  const l = await mustLogin(opts);
  if (!l.apiKey) {
    return (await login()).apiKey;
  }
  return l.apiKey;
};

export const mustGetPrivateKey = async (opts?: LoginOpts) => {
  const l = await mustLogin(opts);
  if (!l.privateKey) {
    return (await login()).privateKey;
  }
  return l.privateKey;
};

export const login = async (opts?: LoginOpts) => {
  const token = await getNewToken(opts);
  credential = await saveLogin(token);
  return credential as Required<Credentials>;
};

interface LoginData {
  privateKey: string;
  apiKey: string;
}

export const saveLogin = async (input: Partial<LoginData>): Promise<Credentials> => {
  const credentialsPath = getCredentialsFilePath();
  const data: Credentials = {
    ...(credential || {}),
    version: '1',
  };
  if (input.privateKey) {
    data.privateKey = input.privateKey;
  }
  if (input.apiKey) {
    data.apiKey = input.apiKey;
  }
  credential = data;
  await outputJSON(credentialsPath, data);
  return data;
};

const CLIENT_ID = 'create-builder';
const PORT = 10110;

const getNewToken = (opts?: LoginOpts) => {
  console.log(`\n🔑 ${yellowBright(bold('Login / Signup required'))}`);

  return askQuestion(
    ` Your browser will open to complete authentication. ${bold('Confirm?')}`
  ).then(next => {
    if (!next) {
      throw new Error('aborted');
    }
    const loading = new Spinner(bold(' Waiting for authorization...'));
    return new Promise<LoginData>((resolve, reject) => {
      const server = http
        .createServer((req, res) => {
          const parsedUrl = new URL(req.url!, 'http://localhost:10110/');
          if (parsedUrl.pathname !== '/auth') {
            reject(new Error('Bad path'));
            return;
          }
          const queryAsObject = parsedUrl.searchParams;
          const privateKey = queryAsObject.get('p-key');
          if (!privateKey) {
            reject(new Error('Missing p-key'));
            return;
          }

          const apiKey = queryAsObject.get('api-key');
          if (!apiKey) {
            reject(new Error('Missing api-key'));
            return;
          }

          const location = opts?.redirectURL ?? `${HOST}/cli-auth?success=true`;
          res.writeHead(302, {
            Location: location,
          });
          res.end();
          req.socket.end();
          req.socket.destroy();
          server.close();
          loading.stop(true);
          resolve({
            privateKey,
            apiKey,
          });
        })
        .listen(PORT);

      loading.setSpinnerString(8);
      loading.start();

      openBuilderAuth(PORT, CLIENT_ID);
    });
  });
};

export const defaultActions = async (options: any) => {
  Object.assign(cliOptions, options);
  if (options.pkey) {
    await saveLogin({
      privateKey: options.pkey,
    });
  }
};

```


## High-Level Overview
JavaScript/TypeScript module. Contains 165 lines of code.

## Detailed Walkthrough

### Functions (9)

- `getCredentialsFilePath()`
- `getLogin()`
- `mustLogin()`
- `mustGetApiKey()`
- `mustGetPrivateKey()`
- `login()`
- `saveLogin()`
- `getNewToken()`
- `defaultActions()`

### Imports (9)

- `cli-spinner`
- `path`
- `./open`
- `./cli`
- `colorette`
- `fs-extra`
- `./utils`
- `http`
- `os`

### Exports (9)

- `defaultActions`
- `mustGetPrivateKey`
- `getCredentialsFilePath`
- `mustLogin`
- `saveLogin`
- `mustGetApiKey`
- `login`
- `getLogin`
- `interface`

## Usage Examples
*No usage examples found in source. Refer to tests or documentation.*

## Dependencies and Related Files
### Imports
- `cli-spinner` (external)
- `path` (external)
- `./open` (local)
- `./cli` (local)
- `colorette` (external)
- `fs-extra` (external)
- `./utils` (local)
- `http` (external)
- `os` (external)

## Performance and Security Notes
*No specific performance or security concerns detected*

## Testing
*No test information available. Check for corresponding test files.*

---
*Generated by Repo Book Generator v1.0.0*
