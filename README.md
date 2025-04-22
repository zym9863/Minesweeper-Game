# Minesweeper Game

[English](README.md) | [中文](README_zh.md)

A modern implementation of the classic Minesweeper game built with Vue 3, TypeScript, and Vite.

## Features

- Classic Minesweeper gameplay
- Multiple difficulty levels (Beginner, Intermediate, Expert)
- Custom game configuration
- Timer and mine counter
- First-click protection (first click is never a mine)
- Modern UI with animations
- Responsive design

## Tech Stack

- **Vue 3** with Composition API and `<script setup>` syntax
- **TypeScript** for type safety
- **Vite** for fast development and optimized builds
- Custom CSS variables for theming

## Project Structure

```
src/
├── assets/         # Icons and images
├── components/     # Vue components
│   ├── Cell.vue    # Individual cell component
│   ├── GameBoard.vue # Main game board
│   └── GameControls.vue # Game controls (timer, restart, difficulty)
├── App.vue         # Main application component
├── main.ts         # Application entry point
└── style.css       # Global styles and CSS variables
```

## How to Play

- Left-click to reveal a cell
- Right-click to place/remove a flag
- Numbers show how many mines are adjacent to that cell
- Flag all mines to win!

## Development

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

## License

MIT

