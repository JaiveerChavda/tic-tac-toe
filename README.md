# Play Tic Tac Toe - Built Using Laravel Reverb

## Description

This is tic tac toe game built with Laravel, Vue.js, and Inertia.js, inspired by the Laracasts course ["Real Time Game with Laravel"](https://laracasts.com/series/real-time-games-with-laravel).  The game allows a player to create a new game, join existing games, and compete against each other in real time. 

This project is a perfect showcasing of [Reverb](https://reverb.laravel.com/) (web-socket server) and [Echo](https://github.com/laravel/echo) (library for listening, event broadcasted by backend server), two powerfull tools mainly used for live-updating user-interfaces. 
If you're begineer and want to learn how modern web-applications update things in real-time then, follow this projects commit step by step to learn every single feature.I tried to Version Control every small thing.

## Features

- **Real-Time Gameplay**: Players can join and create games, allowing for real-time interaction and gameplay.
- **Player Matchups**: A game starts when a second player joins, enabling head-to-head matches.
- **Winner Announcement**: A modal popup announces the winner at the end of each game.

## Upcoming Features

- **Invite Friends**: Invite friends to join the game via email.

## Installation

To get started, clone the repository and follow the steps below:

```bash
# Clone the repository
git clone <https://github.com/JaiveerChavda/tic-tac-toe.git>

# Navigate to the project directory
cd tic-tac-toe

# Install dependencies
composer install
npm install

# Duplicate the .env.example file and rename it to .env
cp .env.example .env

# Generate app key
php artisan generate:key

# Run database migrations
php artisan migrate

# Start the laravel development server
php artisan serve

# Bundle asset/Start hot module replacement 
npm run dev

# Start reverb
php artisan reverb:start

# Listen for queue/Start queue
php artisan queue:work or php artisan queue:listen
```

## Give Feedback 💬

Give your feedback on [@JaiveerChavda](https://x.com/JaiveerChavda)
