# Plaisoram Database Schema

This document represents the current state of the Plaisoram database schema, including the multi-tenancy updates linking Users, Devices, Media, and Playlists to Workspaces.

## Entity-Relationship Diagram

```mermaid
erDiagram
    WORKSPACE {
        int id PK
        string name
        datetime createdAt
        datetime updatedAt
    }
    
    USER {
        int id PK
        string email UK
        string firstName
        string lastName
        string phone "nullable"
        string organization "nullable"
        json roles
        string password
        boolean isActive
        int workspace_id FK
        datetime createdAt
        datetime updatedAt
    }

    REFRESH_TOKEN {
        int id PK
        string refresh_token
        string username
        datetime valid
    }

    DEVICE {
        int id PK
        string name "nullable"
        string pairingCode "nullable"
        int currentPlaylistId "nullable"
        boolean isOnline
        int workspace_id FK "nullable"
        datetime createdAt
        datetime updatedAt
    }

    MEDIA {
        int id PK
        string url
        string type
        int duration
        int workspace_id FK "nullable"
        datetime createdAt
        datetime updatedAt
    }

    PLAYLIST {
        int id PK
        string name
        int workspace_id FK "nullable"
        datetime createdAt
        datetime updatedAt
    }

    PLAYLIST_MEDIA {
        int id PK
        int playlistId FK
        int mediaId FK
        int position
    }

    %% Relationships
    WORKSPACE ||--o{ USER : "owns"
    WORKSPACE ||--o{ DEVICE : "contains"
    WORKSPACE ||--o{ MEDIA : "contains"
    WORKSPACE ||--o{ PLAYLIST : "contains"
    
    PLAYLIST ||--o{ PLAYLIST_MEDIA : "has"
    MEDIA ||--o{ PLAYLIST_MEDIA : "included in"
```

## DBML (Database Markup Language) Definition

```dbml
Table workspace {
  id int [pk, increment]
  name varchar(255)
  createdAt datetime
  updatedAt datetime
}

Table user {
  id int [pk, increment]
  email varchar(180) [unique]
  firstName varchar(255)
  lastName varchar(255)
  phone varchar(255) [null]
  organization varchar(255) [null]
  roles json
  password varchar(255)
  isActive boolean [default: true]
  workspace_id int [not null]
  createdAt datetime
  updatedAt datetime
}

Table refresh_tokens {
  id int [pk, increment]
  refresh_token varchar(128) [unique]
  username varchar(255)
  valid datetime
}

Table device {
  id int [pk, increment]
  name varchar(255) [null]
  pairingCode varchar(255) [null]
  currentPlaylistId int [null]
  isOnline boolean [default: true]
  workspace_id int [null]
  createdAt datetime
  updatedAt datetime
}

Table media {
  id int [pk, increment]
  url varchar(255)
  type varchar(50)
  duration int
  workspace_id int [null]
  createdAt datetime
  updatedAt datetime
}

Table playlist {
  id int [pk, increment]
  name varchar(255)
  workspace_id int [null]
  createdAt datetime
  updatedAt datetime
}

Table playlist_media {
  id int [pk, increment]
  playlistId int
  mediaId int
  position int
}

// Relationships
Ref: user.workspace_id > workspace.id
Ref: device.workspace_id > workspace.id
Ref: media.workspace_id > workspace.id
Ref: playlist.workspace_id > workspace.id
Ref: playlist_media.playlistId > playlist.id
Ref: playlist_media.mediaId > media.id
```
