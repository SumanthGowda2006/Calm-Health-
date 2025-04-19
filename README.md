<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Mood Tracker</title>
    <style>
        body {
            font-family: 'Helvetica Neue', sans-serif;
            background-color: #f7f9fc;
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
        }

        .mood-tracker-container {
            background-color: #fff;
            border-radius: 12px;
            box-shadow: 0 8px 20px rgba(0, 0, 0, 0.08);
            padding: 30px;
            text-align: center;
            width: 90%;
            max-width: 500px;
        }

        h2 {
            color: #33475b;
            margin-bottom: 25px;
            font-weight: 300;
            font-size: 1.8em;
        }

        .mood-selection {
            display: flex;
            gap: 20px;
            justify-content: center;
            margin-bottom: 30px;
        }

        .mood-icon-button {
            background: none;
            border: none;
            cursor: pointer;
            opacity: 0.7;
            transition: opacity 0.2s ease-in-out, transform 0.2s ease-in-out;
            padding: 15px;
            border-radius: 50%;
        }

        .mood-icon-button:hover,
        .mood-icon-button.selected {
            opacity: 1;
            transform: scale(1.1);
        }

        .mood-icon {
            font-size: 2.8em;
            display: block;
        }

        .mood-label {
            display: block;
            margin-top: 8px;
            color: #52616b;
            font-size: 0.9em;
        }

        .mood-happy { color: #fdd835; }
        .mood-calm { color: #80cbc4; }
        .mood-neutral { color: #a7b1bd; }
        .mood-anxious { color: #ffb74d; }
        .mood-sad { color: #90caf9; }

        .mood-visualization {
            margin-top: 40px;
            padding-top: 30px;
            border-top: 1px solid #e0e6ed;
            text-align: left; /* Align suggestions to the left */
        }

        h3 {
            color: #33475b;
            font-weight: 400;
            margin-bottom: 15px;
            font-size: 1.4em;
        }

        #moodHistory {
            color: #52616b;
            font-size: 1em;
            line-height: 1.6;
            margin-bottom: 15px; /* Add some space below the history */
        }

        .mood-streak {
            color: #66bb6a;
            font-size: 1.1em;
            margin-bottom: 20px; /* Add more space before suggestions */
            font-weight: 500;
        }

        .suggestions {
            margin-top: 20px;
        }

        .suggestions h4 {
            color: #33475b;
            font-size: 1.2em;
            margin-bottom: 10px;
            font-weight: 500;
        }

        .suggestions ul {
            list-style: disc;
            padding-left: 20px;
            color: #52616b;
            font-size: 1em;
            line-height: 1.6;
        }

        .suggestions li {
            margin-bottom: 8px;
        }
    </style>
</head>
<body>
    <div class="mood-tracker-container">
        <h2>How are you feeling right now?</h2>
        <div class="mood-selection">
            <button class="mood-icon-button" data-mood="happy">
                <span class="mood-icon mood-happy">😊</span>
                <span class="mood-label">Happy</span>
            </button>
            <button class="mood-icon-button" data-mood="calm">
                <span class="mood-icon mood-calm">😌</span>
                <span class="mood-label">Calm</span>
            </button>
            <button class="mood-icon-button" data-mood="neutral">
                <span class="mood-icon mood-neutral">😐</span>
                <span class="mood-label">Neutral</span>
            </button>
            <button class="mood-icon-button" data-mood="anxious">
                <span class="mood-icon mood-anxious">😟</span>
                <span class="mood-label">Anxious</span>
            </button>
            <button class="mood-icon-button" data-mood="sad">
                <span class="mood-icon mood-sad">😔</span>
                <span class="mood-label">Sad</span>
            </button>
        </div>

        <div class="mood-visualization">
            <h3>Recent Mood</h3>
            <p id="moodHistory">No mood recorded yet.</p>
            <p class="mood-streak" id="streakCounter">Current Streak: 0 days</p>
            <div class="suggestions" id="moodSuggestions">
                </div>
        </div>
    </div>

    <script>
        document.addEventListener('DOMContentLoaded', () => {
            const moodButtons = document.querySelectorAll('.mood-icon-button');
            const moodHistoryDisplay = document.getElementById('moodHistory');
            const streakCounterDisplay = document.getElementById('streakCounter');
            const moodSuggestionsDisplay = document.getElementById('moodSuggestions');

            let moodLog = JSON.parse(localStorage.getItem('moodLog')) || [];

            const moodToActionMap = {
                happy: [
                    "Share your good mood with someone.",
                    "Engage in a creative activity.",
                    "Reflect on what made you happy."
                ],
                calm: [
                    "Continue enjoying the peace.",
                    "Practice mindful breathing.",
                    "Consider a gentle activity like reading."
                ],
                neutral: [
                    "Observe your feelings without judgment.",
                    "Engage in a routine task.",
                    "Check in with your body and mind."
                ],
                anxious: [
                    "Try deep breathing exercises.",
                    "Engage in a calming activity like listening to music.",
                    "Limit caffeine and stimulants.",
                    "Consider talking to someone you trust."
                ],
                sad: [
                    "Allow yourself to feel your emotions.",
                    "Reach out to a friend or family member.",
                    "Engage in gentle self-care activities.",
                    "Listen to calming music or nature sounds."
                ]
            };

            function updateMoodHistoryDisplay() {
                if (moodLog.length === 0) {
                    moodHistoryDisplay.textContent = 'No mood recorded yet.';
                } else {
                    const lastEntry = moodLog[moodLog.length - 1];
                    moodHistoryDisplay.textContent = `Last recorded: ${lastEntry.mood} on ${new Date(lastEntry.timestamp).toLocaleDateString()}`;
                }
            }

            function calculateStreak() {
                if (moodLog.length === 0) {
                    return 0;
                }

                let streak = 0;
                const today = new Date().toDateString();
                let currentDate = new Date(today);
                const sortedLog = [...moodLog].sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp));

                for (const entry of sortedLog) {
                    const entryDate = new Date(entry.timestamp).toDateString();
                    if (entryDate === currentDate) {
                        streak++;
                        currentDate.setDate(currentDate.getDate() - 1);
                    } else if (new Date(entryDate) < currentDate) {
                        break;
                    }
                }
                return streak;
            }

            function updateStreakDisplay() {
                const currentStreak = calculateStreak();
                streakCounterDisplay.textContent = `Current Streak: ${currentStreak} days`;
            }

            function displaySuggestions(mood) {
                const suggestions = moodToActionMap[mood];
                moodSuggestionsDisplay.innerHTML = ''; // Clear previous suggestions

                if (suggestions && suggestions.length > 0) {
                    const suggestionsTitle = document.createElement('h4');
                    suggestionsTitle.textContent = "Things you can do:";
                    moodSuggestionsDisplay.appendChild(suggestionsTitle);

                    const suggestionsList = document.createElement('ul');
                    suggestions.forEach(suggestion => {
                        const listItem = document.createElement('li');
                        listItem.textContent = suggestion;
                        suggestionsList.appendChild(listItem);
                    });
                    moodSuggestionsDisplay.appendChild(suggestionsList);
                }
            }

            moodButtons.forEach(button => {
                button.addEventListener('click', function() {
                    const selectedMood = this.dataset.mood;
                    const timestamp = new Date().toISOString();
                    moodLog.push({ mood: selectedMood, timestamp: timestamp });
                    localStorage.setItem('moodLog', JSON.stringify(moodLog));

                    moodButtons.forEach(btn => btn.classList.remove('selected'));
                    this.classList.add('selected');

                    updateMoodHistoryDisplay();
                    updateStreakDisplay();
                    displaySuggestions(selectedMood); // Show suggestions for the selected mood
                });
            });

            updateMoodHistoryDisplay();
            updateStreakDisplay();

            // Optionally display suggestions for the last recorded mood on load
            if (moodLog.length > 0) {
                displaySuggestions(moodLog[moodLog.length - 1].mood);
            }
        });
    </script>
</body>
</html>
