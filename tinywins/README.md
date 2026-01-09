# Tiny Wins 🏆

**Small steps. Big impact.** Your daily checklist for building better habits.

![Tiny Wins](og-image.png)

## 🌟 Overview

Tiny Wins is a minimalist daily task management app designed to help you build consistent habits through small, achievable daily goals. Unlike traditional to-do apps, Tiny Wins features customizable **template-based routines** for weekdays, weekends, and special days, plus support for **one-time tasks**.

## ✨ Features

### Three-Tab Navigation

- **📅 Today** - Your daily task list with progress tracking
- **📋 Templates** - Customize routines for Weekday, Weekend, and Special Day
- **📜 History** - View past performance and completion rates

### Two Types of Tasks

| Task Type | Visual Style | Behavior |
|-----------|--------------|----------|
| **Routine Tasks** | Warm beige background | From templates, resets daily - perfect for habits |
| **One-Time Tasks** | Light blue background with border | Added on-the-fly, carries over until completed |

### Smart Template System

- **Weekday Template** - Your Monday-Friday routine
- **Weekend Template** - Your Saturday-Sunday routine  
- **Special Day Template** - For holidays, birthdays, or any special occasion
- **Auto-Detection** - Automatically loads Weekday or Weekend based on current day
- **Manual Override** - Click the day mode pill to manually choose any template for today
- **Smart Expiration** - Manual selections apply only to that calendar day
- Choose from pre-built templates or start from scratch

### Core Functionality

- ✅ **Visual Progress** - Real-time progress bar with percentage
- 🔄 **Manual Template Selection** - Click the day mode pill to switch between Weekday, Weekend, or Special Day
- 🎯 **Drag & Drop** - Reorder tasks on Today (temporary) or Templates (permanent)
- 🗑️ **Easy Management** - Delete individual tasks or clear entire templates
- 🎉 **Confetti Celebration** - Rewarded when you complete all tasks
- 💾 **Auto-Save** - Everything persists in browser storage
- 📱 **Mobile First** - Beautiful responsive design
- 🕐 **Timezone Smart** - Uses your local time for accurate day detection

### Smart Behavior

- **Today's Tasks**: Reordering is temporary (for today only)
- **Template Tasks**: Reordering is permanent (saved to template)
- **One-Time Tasks**: Automatically carry over to the next day if uncompleted
- **Manual Override**: Choose any template for today by clicking the day mode pill
  - Applies only to that calendar date
  - Tomorrow automatically reverts to Weekday/Weekend detection
  - Perfect for holidays, vacation days, or any special occasion
- **Reset Today**: Reloads your currently selected template while keeping one-time tasks
- **Day Change Detection**: Automatically loads the correct template when date changes

## 🚀 Getting Started

### Option 1: Use Online
Visit: [https://aprkim.github.io/tiny-wins/](https://aprkim.github.io/tiny-wins/)

### Option 2: Run Locally
1. Clone the repository:
   ```bash
   git clone https://github.com/aprkim/tiny-wins.git
   ```
2. Open `index.html` in your browser

### Add to Home Screen (Mobile)
For the best experience, add Tiny Wins to your phone's home screen:
- **iOS**: Tap Share → Add to Home Screen
- **Android**: Tap Menu → Add to Home Screen

## 🎨 Onboarding Templates

Choose from built-in templates to get started quickly:

- **Daily Productivity** - Morning routine, deep work, planning
- **Health & Wellness** - Exercise, nutrition, hydration
- **Workday Focused** - Inbox management, breaks, task completion
- **Weekend Reset** - Slow morning, family time, house reset
- **Student Mode** - Study sessions, practice, reviews
- **Start from Scratch** - Build your own custom template

Each template initializes your Weekday routine, with a relaxing Weekend template automatically included.

## 🛠️ Technical Details

- **Built with**: Vanilla HTML, CSS, and JavaScript (single-file app)
- **Storage**: Browser LocalStorage
- **Dependencies**: Zero external dependencies
- **Hosting**: GitHub Pages
- **Architecture**: Single Page Application (SPA)

## 📁 Project Structure

```
tiny-wins/
├── index.html         # Complete application (HTML + CSS + JS)
├── favicon.svg        # Browser tab icon
├── og-image.png       # Social media preview image
├── og-image.svg       # Source for preview image
├── test_storage.html  # LocalStorage testing utility
└── README.md          # This file
```

## 💾 Data Storage

All data is stored locally in your browser using these keys:
- `tinyWins_templates` - Your Weekday, Weekend, Special Day templates
- `tinyWins_todayTasks` - Current day's task list
- `tinyWins_todayDate` - Current date tracking
- `tinyWins_todayMode` - Current day mode (weekday/weekend/holiday)
- `tinyWins_todayModeOverride` - Manual template selection (per-date override)
- `tinyWins_history` - Past 30 days of completion data
- `tinyWins_lastActiveTab` - Remembers your last active tab
- `tinyWins_setupComplete` - Onboarding completion flag

## 🎯 Philosophy

Tiny Wins is built on the belief that **consistent small actions lead to significant results**. Instead of overwhelming yourself with ambitious goals, focus on achievable daily wins that compound over time.

> "A journey of a thousand miles begins with a single step." — Lao Tzu

## 🔄 Recent Updates

- **Manual Template Selection** - Click the day mode pill to choose any template for today
- **Color-Matched Modal** - Template selector with pill-themed design
- **Three-Tab Interface** - Streamlined navigation (Today, Templates, History)
- **Template System** - Separate routines for Weekday, Weekend, Special Day
- **Drag & Drop** - Reorder tasks with visual drag handles
- **Timezone Fix** - Accurate local time detection
- **One-Time Tasks** - Blue-tinted cards that carry over until completed
- **Clear All** - Quick way to reset individual templates
- **Improved Mobile UI** - Better touch targets and spacing

## 📝 License

MIT License - Feel free to use, modify, and distribute.

## 👩‍💻 Author

Created by April Kim • 2025

---

**Start your tiny wins journey today!** 🚀
