# Taboosiness

A Vue.js web application for playing the corporate buzzword version of the classic Taboo game.

## 🎮 How to Play

1. **Setup**: Choose 2-5 players and enter their names
2. **Gameplay**: One player tries to make others guess a target word without using the taboo words
3. **Scoring**: 
   - ✅ +1 point for each correct guess
   - ❌ -1 point for mentioning a taboo word
4. **Winning**: The player with the highest score wins!

## 🚀 Getting Started

### Prerequisites
- Node.js (version 14 or higher)
- npm or yarn

### Installation

1. Install dependencies:
```bash
npm install
```

2. Start the development server:
```bash
npm run dev
```

3. Open your browser and navigate to `http://localhost:3000`

### Building for Production

```bash
npm run build
```

The built files will be in the `dist` directory.

## 🎯 Game Features

- **Player Management**: Support for 2-4 players
- **Dynamic Card System**: Random selection from 100+ corporate buzzwords
- **Real-time Scoring**: Track points with visual feedback
- **Responsive Design**: Works on desktop and mobile devices
- **Modern UI**: Beautiful gradient design with smooth animations

## 📝 Word Categories

The game includes corporate buzzwords and their taboo alternatives, perfect for:
- Team building events
- Office parties
- Corporate training sessions
- Language learning (Italian/English business terms)

## 🛠️ Technical Details

- **Framework**: Vue 3 with Composition API
- **Build Tool**: Vite
- **Styling**: Custom CSS with modern design patterns
- **Data**: JSON-based word database

## 📱 Browser Support

- Chrome (recommended)
- Firefox
- Safari
- Edge

## 🎨 Customization

You can easily add more words by editing the `words.json` file. Each word entry should include:
- `id`: Unique identifier
- `targetWord`: The word to be guessed
- `tabooWords`: Array of words that cannot be used as hints

## 📄 License

This project is open source and available under the MIT License.

