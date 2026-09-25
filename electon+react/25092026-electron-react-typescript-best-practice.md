# Electron + React + TypeScript — Best Practice Project Structure

โครงสร้างนี้เหมาะสำหรับ Electron Desktop Application ที่ใช้ React + TypeScript โดยเน้นการแยกความรับผิดชอบระหว่าง:

- Electron Main Process
- Preload
- React Renderer
- Shared Types / IPC Contracts

---

## Recommended Folder Structure

```text
my-electron-app/
├── package.json
├── package-lock.json
├── tsconfig.json
├── tsconfig.node.json
├── vite.config.ts
├── electron-builder.yml
├── .env
├── .env.production
├── .gitignore
│
├── assets/
│   ├── icon.icns
│   ├── icon.ico
│   └── icon.png
│
├── resources/
│   ├── bridge.sh
│   └── templates/
│
├── src/
│   │
│   ├── main/
│   │   ├── index.ts
│   │   │
│   │   ├── windows/
│   │   │   ├── mainWindow.ts
│   │   │   ├── settingsWindow.ts
│   │   │   └── notificationWindow.ts
│   │   │
│   │   ├── ipc/
│   │   │   ├── index.ts
│   │   │   ├── terminal.ipc.ts
│   │   │   ├── settings.ipc.ts
│   │   │   └── system.ipc.ts
│   │   │
│   │   ├── services/
│   │   │   ├── terminal.service.ts
│   │   │   ├── notification.service.ts
│   │   │   ├── storage.service.ts
│   │   │   └── updater.service.ts
│   │   │
│   │   ├── managers/
│   │   │   ├── window.manager.ts
│   │   │   └── tray.manager.ts
│   │   │
│   │   └── utils/
│   │       ├── logger.ts
│   │       └── paths.ts
│   │
│   ├── preload/
│   │   ├── index.ts
│   │   └── api/
│   │       ├── terminal.api.ts
│   │       ├── settings.api.ts
│   │       └── system.api.ts
│   │
│   ├── renderer/
│   │   ├── main.tsx
│   │   ├── App.tsx
│   │   │
│   │   ├── pages/
│   │   │   ├── HomePage.tsx
│   │   │   └── SettingsPage.tsx
│   │   │
│   │   ├── features/
│   │   │   ├── sessions/
│   │   │   │   ├── components/
│   │   │   │   │   ├── SessionList.tsx
│   │   │   │   │   └── SessionCard.tsx
│   │   │   │   ├── hooks/
│   │   │   │   │   └── useSessions.ts
│   │   │   │   ├── services/
│   │   │   │   │   └── session.service.ts
│   │   │   │   ├── types.ts
│   │   │   │   └── index.ts
│   │   │   │
│   │   │   ├── notifications/
│   │   │   └── settings/
│   │   │
│   │   ├── components/
│   │   │   ├── common/
│   │   │   └── layout/
│   │   │
│   │   ├── hooks/
│   │   ├── store/
│   │   ├── utils/
│   │   ├── constants/
│   │   ├── types/
│   │   │   └── global.d.ts
│   │   └── styles/
│   │
│   └── shared/
│       ├── ipc/
│       │   ├── channels.ts
│       │   ├── contracts.ts
│       │   └── types.ts
│       │
│       ├── types/
│       │   ├── session.ts
│       │   └── settings.ts
│       │
│       ├── schemas/
│       └── constants/
│
├── scripts/
│   ├── build.ts
│   └── afterSign.ts
│
├── tests/
│   ├── main/
│   └── renderer/
│
└── dist/
```

---

# Architecture

```text
React Component
      ↓
Feature Hook / Feature Service
      ↓
window.electronAPI
      ↓
Preload
      ↓
IPC
      ↓
Main IPC Handler
      ↓
Main Service
      ↓
OS / File System / Terminal / External Process
```

---

# Folder Responsibilities

| Folder | Responsibility |
|---|---|
| `src/main` | Electron Main Process |
| `src/main/windows` | BrowserWindow creation and configuration |
| `src/main/ipc` | IPC request handlers |
| `src/main/services` | Business logic / OS integration |
| `src/main/managers` | Global managers เช่น Window, Tray |
| `src/preload` | Secure bridge ระหว่าง Renderer และ Main |
| `src/renderer` | React application |
| `src/renderer/features` | Feature-based frontend modules |
| `src/shared` | Types, IPC contracts และ constants ที่ใช้ร่วมกัน |

---

# Shared IPC Channels

ไม่ควรเขียน IPC channel เป็น string ซ้ำกระจายทั่ว project

```ts
// src/shared/ipc/channels.ts

export const IPC_CHANNELS = {
  TERMINAL: {
    GET_SESSIONS: 'terminal:get-sessions',
    OPEN_SESSION: 'terminal:open-session',
  },

  SETTINGS: {
    GET: 'settings:get',
    SAVE: 'settings:save',
  },
} as const
```

---

# Shared Type

```ts
// src/shared/types/session.ts

export interface Session {
  id: string
  name: string
  status: 'active' | 'idle' | 'waiting'
  createdAt: string
}
```

---

# IPC Contracts

แนะนำให้กำหนด Request / Response ของ IPC ไว้ใน shared layer

```ts
// src/shared/ipc/contracts.ts

import type { Session } from '../types/session'

export interface GetSessionsResponse {
  sessions: Session[]
}

export interface OpenSessionRequest {
  sessionId: string
}

export interface OpenSessionResponse {
  success: boolean
}
```

ข้อดีคือ Main, Preload และ Renderer ใช้ contract เดียวกัน

---

# Electron Main IPC Handler

```ts
// src/main/ipc/terminal.ipc.ts

import { ipcMain } from 'electron'
import { IPC_CHANNELS } from '../../shared/ipc/channels'
import { terminalService } from '../services/terminal.service'

export function registerTerminalIPC(): void {
  ipcMain.handle(
    IPC_CHANNELS.TERMINAL.GET_SESSIONS,
    async () => {
      return terminalService.getSessions()
    }
  )

  ipcMain.handle(
    IPC_CHANNELS.TERMINAL.OPEN_SESSION,
    async (_event, sessionId: string) => {
      return terminalService.openSession(sessionId)
    }
  )
}
```

---

# Main Service

Business logic ควรแยกออกจาก IPC Handler

```ts
// src/main/services/terminal.service.ts

import type { Session } from '../../shared/types/session'

class TerminalService {
  async getSessions(): Promise<Session[]> {
    return []
  }

  async openSession(sessionId: string): Promise<boolean> {
    console.log('Opening session:', sessionId)

    return true
  }
}

export const terminalService = new TerminalService()
```

IPC Handler มีหน้าที่รับ request

Service มีหน้าที่ทำงานจริง

---

# Preload API

Renderer ไม่ควรเข้าถึง `ipcRenderer` โดยตรง

```ts
// src/preload/index.ts

import { contextBridge, ipcRenderer } from 'electron'
import { IPC_CHANNELS } from '../shared/ipc/channels'
import type { Session } from '../shared/types/session'

const electronAPI = {
  terminal: {
    getSessions: (): Promise<Session[]> =>
      ipcRenderer.invoke(
        IPC_CHANNELS.TERMINAL.GET_SESSIONS
      ),

    openSession: (
      sessionId: string
    ): Promise<boolean> =>
      ipcRenderer.invoke(
        IPC_CHANNELS.TERMINAL.OPEN_SESSION,
        sessionId
      ),
  },
}

contextBridge.exposeInMainWorld(
  'electronAPI',
  electronAPI
)

export type ElectronAPI = typeof electronAPI
```

---

# Global Type Declaration

เพื่อให้ React รู้จัก `window.electronAPI`

```ts
// src/renderer/types/global.d.ts

import type { ElectronAPI } from '../../preload'

declare global {
  interface Window {
    electronAPI: ElectronAPI
  }
}

export {}
```

จากนั้น React จะได้ TypeScript autocomplete เช่น

```ts
window.electronAPI.terminal.getSessions()
```

---

# React Usage

```tsx
import { useEffect, useState } from 'react'
import type { Session } from '@shared/types/session'

export function SessionList() {
  const [sessions, setSessions] = useState<Session[]>([])

  useEffect(() => {
    window.electronAPI.terminal
      .getSessions()
      .then(setSessions)
  }, [])

  return (
    <div>
      {sessions.map((session) => (
        <div key={session.id}>
          {session.name}
        </div>
      ))}
    </div>
  )
}
```

---

# Feature-Based Renderer Structure

React project ที่เริ่มใหญ่ไม่ควรแยกทุกอย่างไว้แค่

```text
components/
hooks/
services/
```

เพราะแต่ละ feature จะกระจายอยู่หลาย folder

แนะนำ Feature-Based Structure

```text
renderer/
└── features/
    ├── sessions/
    │   ├── components/
    │   │   ├── SessionList.tsx
    │   │   └── SessionCard.tsx
    │   ├── hooks/
    │   │   └── useSessions.ts
    │   ├── services/
    │   │   └── session.service.ts
    │   ├── types.ts
    │   └── index.ts
    │
    ├── approvals/
    │   ├── components/
    │   ├── hooks/
    │   └── services/
    │
    ├── notifications/
    │   ├── components/
    │   └── hooks/
    │
    └── settings/
        ├── components/
        ├── hooks/
        └── settings.store.ts
```

Feature ไหนแก้ Feature นั้น

---

# Recommended TypeScript Configuration

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "Bundler",

    "strict": true,

    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "resolveJsonModule": true,
    "skipLibCheck": true,

    "baseUrl": ".",

    "paths": {
      "@main/*": ["src/main/*"],
      "@renderer/*": ["src/renderer/*"],
      "@shared/*": ["src/shared/*"]
    }
  }
}
```

---

# Security Best Practices

Electron Renderer ไม่ควรใช้ Node.js API โดยตรง

ไม่ควร:

```ts
const { ipcRenderer } = require('electron')
```

ไม่ควร:

```ts
const fs = require('fs')
```

ไม่ควร expose:

```ts
contextBridge.exposeInMainWorld(
  'electron',
  ipcRenderer
)
```

ควร expose เฉพาะ API ที่ Renderer จำเป็นต้องใช้

```ts
contextBridge.exposeInMainWorld(
  'electronAPI',
  {
    terminal: {
      getSessions: () =>
        ipcRenderer.invoke('terminal:get-sessions')
    }
  }
)
```

แนะนำ BrowserWindow configuration:

```ts
new BrowserWindow({
  webPreferences: {
    preload: preloadPath,

    contextIsolation: true,
    nodeIntegration: false,
    sandbox: true,
  },
})
```

---

# Recommended VibeNotis Structure

สำหรับ Application ที่มี Session, Terminal, Approval, Question, Prompt และ Notification สามารถจัดแบบนี้

```text
src/
├── main/
│   ├── index.ts
│   │
│   ├── windows/
│   │   ├── notchWindow.ts
│   │   └── settingsWindow.ts
│   │
│   ├── ipc/
│   │   ├── session.ipc.ts
│   │   ├── terminal.ipc.ts
│   │   ├── approval.ipc.ts
│   │   └── settings.ipc.ts
│   │
│   ├── services/
│   │   ├── claude.service.ts
│   │   ├── terminal.service.ts
│   │   ├── bridge.service.ts
│   │   └── notification.service.ts
│   │
│   └── managers/
│       ├── window.manager.ts
│       └── permission.manager.ts
│
├── preload/
│   ├── index.ts
│   └── api/
│       ├── session.api.ts
│       ├── terminal.api.ts
│       ├── approval.api.ts
│       └── settings.api.ts
│
├── renderer/
│   ├── main.tsx
│   ├── App.tsx
│   │
│   ├── features/
│   │   ├── sessions/
│   │   ├── approvals/
│   │   ├── questions/
│   │   ├── prompts/
│   │   └── settings/
│   │
│   ├── components/
│   ├── store/
│   ├── hooks/
│   ├── utils/
│   └── styles/
│
└── shared/
    ├── ipc/
    │   ├── channels.ts
    │   └── contracts.ts
    │
    └── types/
        ├── session.ts
        ├── approval.ts
        ├── question.ts
        └── settings.ts
```

---

# Recommended Design Rules

1. **Renderer = UI only**
   - React
   - State
   - UI logic

2. **Preload = Security Boundary**
   - expose เฉพาะ API ที่จำเป็น

3. **IPC Handler = Controller**
   - รับ Request
   - Validate
   - เรียก Service

4. **Service = Business Logic**
   - Terminal
   - File System
   - Shell
   - External Process
   - Database

5. **Shared = Contract**
   - Types
   - Schemas
   - IPC Channels
   - Request / Response contracts

6. **Feature-Based React**
   - แยกตาม Business Feature
   - ไม่แยกแค่ตาม technical type

7. **Avoid `any`**
   - ใช้ TypeScript strict mode

8. **Do not expose Node API to Renderer**

---

# Final Architecture

```text
┌───────────────────────────────┐
│        React Renderer         │
│                               │
│ Component                     │
│    ↓                          │
│ Hook / Feature Service        │
│    ↓                          │
│ window.electronAPI            │
└──────────────┬────────────────┘
               │
               ↓
┌───────────────────────────────┐
│            Preload            │
│                               │
│ contextBridge                 │
│ ipcRenderer.invoke()          │
└──────────────┬────────────────┘
               │
               │ IPC
               ↓
┌───────────────────────────────┐
│        Electron Main          │
│                               │
│ IPC Handler                   │
│    ↓                          │
│ Service                       │
│    ↓                          │
│ Manager / OS Integration      │
└──────────────┬────────────────┘
               │
               ↓
       ┌───────────────┐
       │      OS       │
       │               │
       │ File System   │
       │ Terminal      │
       │ Shell         │
       │ Notification  │
       │ External Apps │
       └───────────────┘
```

---

## Summary

สำหรับ Electron + React + TypeScript ที่ต้องการ maintainability และ scalability:

```text
Main
↓
IPC Handler
↓
Service

Renderer
↓
Feature
↓
Preload API
↓
IPC

Shared
↓
Types
Contracts
Channels
Schemas
```

Structure นี้เหมาะกับ project ตั้งแต่ขนาดกลางจนถึง production desktop application และช่วยให้ Electron security boundary ชัดเจน รวมถึงรักษา Type Safety ตั้งแต่ React ไปจนถึง Electron Main Process
