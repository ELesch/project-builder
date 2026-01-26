# Project Brief: BigBadge - AI-Powered Name Tag Generator

## Project Details

- **Project Name**: BigBadge
- **Repository**: big-badge (GitHub)
- **Deployment**: Vercel
- **Database Hosting**: Supabase

## Overview

A web application that generates personalized name tags on-demand at conferences and events using AI image generation. Instead of pre-printing generic name tags with small text sized for the longest possible name, this app creates optimized, readable name tags when attendees check in, using Google's Nano Banana AI to maximize the visual impact of each person's name.

## Purpose

- **Problem**: Traditional printed name tags waste space - names are too small because they must accommodate the longest possible name and company. This makes them hard to read at a glance.
- **Goal**: Create dynamic, AI-generated name tags that efficiently use the available space, with the first name being the most prominent and readable element.

## Users

- **Primary users**: Conference/event attendees (self-service check-in on tablets/PCs)
- **Administrators**: Event organizers (template design, data import, system setup)
- **Scale**: Large events with hundreds of attendees, multiple simultaneous check-in devices

## Core Features

### 1. Attendee Check-in
- Self-service interface on tablets/PCs
- Search for name in pre-registered attendee list
- On-the-spot registration for walk-ins
- Match attendee to Excel import data

### 2. Template Designer (Event Organizer Tool)
- Specify overall badge/label dimensions
- Designate the area for AI-generated image
- Add static elements (logos, images, text)
- Support `${fieldname}` placeholder syntax for dynamic content
- Map placeholders to Excel column headers

### 3. AI Image Generation
- Integration with Google Nano Banana (Gemini API)
- Generate optimized name graphics:
  - First name: largest/most prominent
  - Last name: secondary
  - Company name: tertiary
- Use entire designated image area efficiently
- Abstraction layer for future AI provider flexibility

### 4. Data Import
- Import attendee list from Excel files
- First row contains field names (column headers)
- Map fields to template placeholders

### 5. Printing
- Direct printing to label printers from the app
- Support common brands: Dymo, Brother, Zebra
- Print immediately after generation

### 6. Offline Fallback
- When Nano Banana is unavailable, generate plain-text badges
- Use template layout with `${fieldname}` substitution
- Ensures event can continue without AI dependency

## Tech Stack

- **Language**: TypeScript
- **Framework**: Next.js
- **Platform**: Web application (browser-based)
- **Architecture**: Central server with multi-device client support
- **Database**: PostgreSQL
- **API Integration**: Google Gemini API (Nano Banana image generation)
  - REST endpoint: `https://generativelanguage.googleapis.com/v1beta/models/{model-id}:generateContent`
  - Authentication: API key (`x-goog-api-key` header)
  - Supported aspect ratios: 1:1, 2:3, 3:2, 3:4, 4:3, 4:5, 5:4, 9:16, 16:9, 21:9
- **Default Label Size**: 4" x 3" (designer supports custom sizes)
- **Hosting**: Vercel (frontend/API) + Supabase (PostgreSQL database)
- **Source Control**: GitHub (repository: big-badge)

## Requirements

### Authentication
- Event organizer login for admin functions
- Basic authentication (not high-security)
- Check-in interface is public-facing (no attendee login)

### Multi-Device Support
- Central server architecture
- Real-time sync across multiple check-in stations
- Internet connectivity required (for AI generation)

### Printing Integration
- Direct browser-to-printer capability
- Support for common label printer protocols
- Configurable for different label sizes

## Constraints

- Requires internet connectivity for AI image generation
- Dependent on Google Nano Banana API availability and pricing (~$0.15/image)
- Label printer compatibility may vary by browser/OS

## Open Questions

None - all requirements captured.

---

Created: 2026-01-22
Updated: 2026-01-22
Status: Complete
