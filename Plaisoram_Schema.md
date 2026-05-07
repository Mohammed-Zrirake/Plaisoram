# Plaisoram Database Schema

This document represents the current state of the Plaisoram database schema, including the multi-tenancy updates linking Users, Devices, Media, and Playlists to Workspaces, as well as the new layout/zone structures.

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
        int resolutionWidth "nullable"
        int resolutionHeight "nullable"
        int workspace_id FK "nullable"
        datetime createdAt
        datetime updatedAt
    }

    MEDIA_FOLDER {
        int id PK
        string name
        string description "nullable"
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
        int folder_id FK "nullable"
        datetime createdAt
        datetime updatedAt
    }

    PLAYLIST {
        int id PK
        string name
        string layoutType "nullable"
        int resolutionX "nullable"
        int resolutionY "nullable"
        int workspace_id FK "nullable"
        datetime createdAt
        datetime updatedAt
    }

    ZONE {
        int id PK
        string name
        int playlist_id FK
        float xPercent
        float yPercent
        float widthPercent
        float heightPercent
        string zoneKey "nullable"
        int mediaId FK "nullable"
    }

    PLAYLIST_MEDIA {
        int id PK
        int zoneId FK
        int mediaId FK
        int position
    }

    %% Relationships
    WORKSPACE ||--o{ USER : "owns"
    WORKSPACE ||--o{ DEVICE : "contains"
    WORKSPACE ||--o{ MEDIA : "contains"
    WORKSPACE ||--o{ PLAYLIST : "contains"
    WORKSPACE ||--o{ MEDIA_FOLDER : "contains"
    
    MEDIA_FOLDER ||--o{ MEDIA : "contains"
    PLAYLIST ||--o{ ZONE : "has"
    ZONE ||--o{ PLAYLIST_MEDIA : "has"
    MEDIA ||--o{ PLAYLIST_MEDIA : "included in"
    MEDIA ||--o{ ZONE : "included in"
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
  resolutionWidth int [null]
  resolutionHeight int [null]
  workspace_id int [null]
  createdAt datetime
  updatedAt datetime
}

Table media_folder {
  id int [pk, increment]
  name varchar(255)
  description varchar(255) [null]
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
  folder_id int [null]
  createdAt datetime
  updatedAt datetime
}

Table playlist {
  id int [pk, increment]
  name varchar(255)
  layoutType varchar(255) [null]
  resolutionX int [null]
  resolutionY int [null]
  workspace_id int [null]
  createdAt datetime
  updatedAt datetime
}

Table zone {
  id int [pk, increment]
  name varchar(255)
  playlist_id int [not null]
  xPercent float
  yPercent float
  widthPercent float
  heightPercent float
  zoneKey varchar(100) [null]
  mediaId int [null]
}

Table playlist_media {
  id int [pk, increment]
  zoneId int
  mediaId int
  position int
}

// Relationships
Ref: user.workspace_id > workspace.id
Ref: device.workspace_id > workspace.id
Ref: media_folder.workspace_id > workspace.id
Ref: media.workspace_id > workspace.id
Ref: media.folder_id > media_folder.id
Ref: playlist.workspace_id > workspace.id
Ref: zone.playlist_id > playlist.id
Ref: zone.mediaId > media.id
Ref: playlist_media.zoneId > zone.id
Ref: playlist_media.mediaId > media.id
```
