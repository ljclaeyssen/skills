# SPA Integration Patterns

Patterns for integrating the Blizzard API into Angular, React, and Vue applications. The key challenge is token management — the OAuth token is server-side only (client_secret must never be exposed to the browser).

## Architecture

```
Browser (Angular/React/Vue)
  ↓ API calls
Your Backend (NestJS/Express/etc.)
  ↓ Blizzard API calls with Bearer token
Blizzard API
```

The browser never calls Blizzard directly — your backend proxies the requests and manages the OAuth token.

## Backend: Token Management Service

### NestJS Example

```typescript
import { Injectable, Logger } from '@nestjs/common';

@Injectable()
export class BlizzardTokenService {
  private token: string | null = null;
  private expiresAt = 0;
  private readonly logger = new Logger(BlizzardTokenService.name);

  constructor(
    private readonly clientId: string,
    private readonly clientSecret: string,
  ) {}

  async getToken(): Promise<string> {
    // Refresh 60s before expiry
    if (this.token && Date.now() < this.expiresAt - 60_000) {
      return this.token;
    }
    return this.refreshToken();
  }

  private async refreshToken(): Promise<string> {
    const response = await fetch('https://oauth.battle.net/token', {
      method: 'POST',
      headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
      body: new URLSearchParams({
        grant_type: 'client_credentials',
        client_id: this.clientId,
        client_secret: this.clientSecret,
      }),
    });

    if (!response.ok) {
      throw new Error(`Blizzard OAuth failed: ${response.status}`);
    }

    const data = await response.json();
    this.token = data.access_token;
    this.expiresAt = Date.now() + data.expires_in * 1000;
    this.logger.log('Blizzard token refreshed');
    return this.token!;
  }
}
```

### Express Example

```typescript
let token: string | null = null;
let expiresAt = 0;

async function getBlizzardToken(): Promise<string> {
  if (token && Date.now() < expiresAt - 60_000) return token;

  const res = await fetch('https://oauth.battle.net/token', {
    method: 'POST',
    headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
    body: new URLSearchParams({
      grant_type: 'client_credentials',
      client_id: process.env.BLIZZARD_CLIENT_ID!,
      client_secret: process.env.BLIZZARD_CLIENT_SECRET!,
    }),
  });

  const data = await res.json();
  token = data.access_token;
  expiresAt = Date.now() + data.expires_in * 1000;
  return token!;
}
```

## Backend: API Proxy Service

### NestJS Example

```typescript
import { Injectable } from '@nestjs/common';

@Injectable()
export class BlizzardApiService {
  private readonly baseUrl = 'https://eu.api.blizzard.com';

  constructor(private readonly tokenService: BlizzardTokenService) {}

  async get<T>(path: string, namespace: string, locale = 'en_US'): Promise<T> {
    const token = await this.tokenService.getToken();
    const url = new URL(path, this.baseUrl);
    url.searchParams.set('namespace', namespace);
    url.searchParams.set('locale', locale);

    const response = await fetch(url.toString(), {
      headers: { Authorization: `Bearer ${token}` },
    });

    // On 401, refresh token and retry once
    if (response.status === 401) {
      const newToken = await this.tokenService.refreshToken();
      const retry = await fetch(url.toString(), {
        headers: { Authorization: `Bearer ${newToken}` },
      });
      return retry.json();
    }

    if (!response.ok) {
      throw new Error(`Blizzard API ${response.status}: ${path}`);
    }

    return response.json();
  }

  // Convenience methods
  getCharacter(realm: string, name: string) {
    return this.get(`/profile/wow/character/${realm}/${name.toLowerCase()}`, 'profile-eu');
  }

  getCharacterEquipment(realm: string, name: string) {
    return this.get(`/profile/wow/character/${realm}/${name.toLowerCase()}/equipment`, 'profile-eu');
  }

  getGuildRoster(realm: string, guild: string) {
    return this.get(`/data/wow/guild/${realm}/${guild.toLowerCase()}/roster`, 'profile-eu');
  }
}
```

## Backend: Controller Example

### NestJS

```typescript
import { Controller, Get, Param } from '@nestjs/common';

@Controller('api/blizzard')
export class BlizzardController {
  constructor(private readonly blizzardApi: BlizzardApiService) {}

  @Get('character/:realm/:name')
  getCharacter(@Param('realm') realm: string, @Param('name') name: string) {
    return this.blizzardApi.getCharacter(realm, name);
  }

  @Get('character/:realm/:name/equipment')
  getEquipment(@Param('realm') realm: string, @Param('name') name: string) {
    return this.blizzardApi.getCharacterEquipment(realm, name);
  }

  @Get('guild/:realm/:name/roster')
  getGuildRoster(@Param('realm') realm: string, @Param('name') name: string) {
    return this.blizzardApi.getGuildRoster(realm, name);
  }
}
```

## Frontend: Angular Service

```typescript
import { Injectable, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({ providedIn: 'root' })
export class BlizzardService {
  private readonly http = inject(HttpClient);
  private readonly apiBase = '/api/blizzard';

  getCharacter(realm: string, name: string): Observable<BlizzardCharacterProfile> {
    return this.http.get<BlizzardCharacterProfile>(
      `${this.apiBase}/character/${realm}/${name.toLowerCase()}`
    );
  }

  getEquipment(realm: string, name: string): Observable<BlizzardCharacterEquipment> {
    return this.http.get<BlizzardCharacterEquipment>(
      `${this.apiBase}/character/${realm}/${name.toLowerCase()}/equipment`
    );
  }

  getGuildRoster(realm: string, guild: string): Observable<BlizzardGuildRoster> {
    return this.http.get<BlizzardGuildRoster>(
      `${this.apiBase}/guild/${realm}/${guild.toLowerCase()}/roster`
    );
  }
}
```

## Frontend: React Hook

```typescript
import { useState, useEffect } from 'react';

function useBlizzardApi<T>(path: string) {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    setLoading(true);
    fetch(`/api/blizzard${path}`)
      .then(res => {
        if (!res.ok) throw new Error(`API error: ${res.status}`);
        return res.json();
      })
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [path]);

  return { data, loading, error };
}

// Usage
function CharacterProfile({ realm, name }: { realm: string; name: string }) {
  const { data, loading } = useBlizzardApi<BlizzardCharacterProfile>(
    `/character/${realm}/${name.toLowerCase()}`
  );
  if (loading) return <div>Loading...</div>;
  return <div>{data?.name} - {data?.equipped_item_level} ilvl</div>;
}
```

## Raider.io (Direct from Browser)

Raider.io requires no auth and allows CORS — call it directly from the browser.

```typescript
// Angular
@Injectable({ providedIn: 'root' })
export class RaiderIoService {
  private readonly http = inject(HttpClient);
  private readonly baseUrl = 'https://raider.io/api/v1';

  getCharacter(region: string, realm: string, name: string, fields = 'mythic_plus_scores_by_season:current,gear') {
    return this.http.get<RaiderIoCharacterProfile>(`${this.baseUrl}/characters/profile`, {
      params: { region, realm, name: name.toLowerCase(), fields }
    });
  }
}

// React
async function getRaiderIoProfile(region: string, realm: string, name: string) {
  const params = new URLSearchParams({
    region, realm, name: name.toLowerCase(),
    fields: 'mythic_plus_scores_by_season:current,gear'
  });
  const res = await fetch(`https://raider.io/api/v1/characters/profile?${params}`);
  return res.json();
}
```

## Environment Variables

```env
# .env
BLIZZARD_CLIENT_ID=your-client-id
BLIZZARD_CLIENT_SECRET=your-client-secret
```

Never expose these to the frontend. They stay on the backend only.
