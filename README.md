
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vitality Coach - Personalized Wellness Plan</title>
    
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Google Fonts: Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap" rel="stylesheet">
    
    <!-- FontAwesome for icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <!-- React & ReactDOM -->
    <script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    
    <!-- Babel for JSX compilation -->
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>

    <script>
        tailwind.config = {
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    },
                    colors: {
                        primary: {
                            50: '#f0fdfa',
                            100: '#ccfbf1',
                            200: '#99f6e4',
                            300: '#5eead4',
                            400: '#2dd4bf',
                            500: '#14b8a6',
                            600: '#0d9488',
                            700: '#0f766e',
                            800: '#115e59',
                            900: '#134e4a',
                        }
                    }
                }
            }
        }
    </script>
    <style>
        body {
            background-color: #f0fdfa; /* primary-50 */
            color: #134e4a; /* primary-900 */
        }
        /* Custom scrollbar for a cleaner look */
        ::-webkit-scrollbar {
            width: 8px;
        }
        ::-webkit-scrollbar-track {
            background: transparent;
        }
        ::-webkit-scrollbar-thumb {
            background: #99f6e4;
            border-radius: 4px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #2dd4bf;
        }
        /* Animation for the rotating message */
        @keyframes fade-in-up {
            0% { opacity: 0; transform: translateY(10px); }
            100% { opacity: 1; transform: translateY(0); }
        }
        .animate-fade-in-up {
            animation: fade-in-up 0.5s ease-out forwards;
        }
    </style>
</head>
<body class="antialiased min-h-screen flex flex-col">
    <div id="root" class="flex-grow flex flex-col"></div>

    <script type="text/babel">
        const { useState, useEffect, useRef } = React;

        const FALLBACK_PLAN = {
            activities: [
                "Take a 20-minute daily walk in nature.",
                "Practice deep breathing exercises for 5 minutes twice a day.",
                "Do some light stretching every morning to improve flexibility."
            ],
            diet: [
                "Ensure you drink at least 8 glasses of water daily.",
                "Include one extra serving of leafy green vegetables with dinner.",
                "Swap out sugary snacks for fresh fruit or nuts."
            ],
            messages: [
                "Every small step you take is progress towards a healthier you.",
                "Listen to your body today; rest if you need to, push if you can.",
                "You are capable of building wonderful, healthy habits.",
                "Nourish your body and mind—you deserve it.",
                "Take it one day at a time. Consistency is key."
            ]
        };

        async function generateWellnessPlan(age, physical, mental) {
            const prompt = `Act as an expert health and wellness coach. 
            I am ${age} years old. 
            My current physical health is: ${physical}. 
            My current mental health is: ${mental}.
            
            Provide a personalized wellness plan. Focus on realistic, actionable advice suitable for this profile.
            Provide exactly 3-5 specific healthcare/physical activities.
            Provide exactly 3-5 specific dietary recommendations.
            Provide exactly 5 short, encouraging, personalized motivational messages that I can read throughout the day.`;

            const payload = {
                contents: [{ parts: [{ text: prompt }] }],
                generationConfig: {
                    responseMimeType: "application/json",
                    responseSchema: {
                        type: "OBJECT",
                        properties: {
                            "activities": { "type": "ARRAY", "items": { "type": "STRING" } },
                            "diet": { "type": "ARRAY", "items": { "type": "STRING" } },
                            "messages": { "type": "ARRAY", "items": { "type": "STRING" } }
                        },
                        required: ["activities", "diet", "messages"]
                    }
                }
            };

            const apiKey = ""; // Canvas handles this
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent?key=${apiKey}`;

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                
                const result = await response.json();
                
                if (result.candidates && result.candidates.length > 0 &&
                    result.candidates[0].content && result.candidates[0].content.parts &&
                    result.candidates[0].content.parts.length > 0) {
                    
                    const jsonString = result.candidates[0].content.parts[0].text;
                    const parsedJson = JSON.parse(jsonString);
                    return parsedJson;
                } else {
                    console.warn("Unexpected API response structure, using fallback.");
                    return FALLBACK_PLAN;
                }
            } catch (error) {
                console.error("Error generating plan:", error);
                // In a real app we might retry, but here we'll fall back gracefully
                return FALLBACK_PLAN;
            }
        }

        const InputForm = ({ onSubmit }) => {
            const [age, setAge] = useState('');
            const [physical, setPhysical] = useState('Average');
            const [mental, setMental] = useState('Neutral');
            const [error, setError] = useState('');

            const handleSubmit = (e) => {
                e.preventDefault();
                if (!age || age <= 0 || age > 120) {
                    setError("Please enter a valid age.");
                    return;
                }
                setError('');
                onSubmit({ age, physical, mental });
            };

            return (
                <div class="max-w-md w-full mx-auto bg-white rounded-2xl shadow-xl overflow-hidden mt-10">
                    <div class="bg-primary-600 p-6 text-white text-center">
                        <i class="fa-solid fa-leaf text-4xl mb-2"></i>
                        <h2 class="text-2xl font-bold">Your Wellness Journey</h2>
                        <p class="text-primary-100 text-sm mt-1">Tell us a bit about yourself to get started.</p>
                    </div>
                    <form onSubmit={handleSubmit} class="p-6 space-y-6">
                        {error && <div class="bg-red-50 text-red-600 p-3 rounded-lg text-sm">{error}</div>}
                        
                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Age</label>
                            <input 
                                type="number" 
                                value={age}
                                onChange={(e) => setAge(e.target.value)}
                                placeholder="e.g., 35"
                                class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-primary-500 focus:border-primary-500 outline-none transition-all"
                            />
                        </div>

                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Current Physical Health</label>
                            <select 
                                value={physical}
                                onChange={(e) => setPhysical(e.target.value)}
                                class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-primary-500 focus:border-primary-500 outline-none transition-all appearance-none bg-white"
                            >
                                <option value="Poor (Struggling with daily tasks, chronic pain)">Poor</option>
                                <option value="Below Average (Occasional issues, low energy)">Below Average</option>
                                <option value="Average (Generally okay, could be fitter)">Average</option>
                                <option value="Good (Active, feel healthy most days)">Good</option>
                                <option value="Excellent (Very active, high energy, athletic)">Excellent</option>
                            </select>
                        </div>

                        <div>
                            <label class="block text-sm font-medium text-gray-700 mb-1">Current Mental Health</label>
                            <select 
                                value={mental}
                                onChange={(e) => setMental(e.target.value)}
                                class="w-full px-4 py-2 border border-gray-300 rounded-lg focus:ring-2 focus:ring-primary-500 focus:border-primary-500 outline-none transition-all appearance-none bg-white"
                            >
                                <option value="Stressed / Overwhelmed">Stressed / Overwhelmed</option>
                                <option value="Anxious / Worried">Anxious / Worried</option>
                                <option value="Neutral / Okay">Neutral / Okay</option>
                                <option value="Good / Generally Positive">Good / Generally Positive</option>
                                <option value="Excellent / Thriving">Excellent / Thriving</option>
                            </select>
                        </div>

                        <button 
                            type="submit"
                            class="w-full bg-primary-600 hover:bg-primary-700 text-white font-semibold py-3 px-4 rounded-lg transition-colors duration-200 flex items-center justify-center gap-2"
                        >
                            Generate My Plan <i class="fa-solid fa-wand-magic-sparkles"></i>
                        </button>
                    </form>
                </div>
            );
        };

        const LoadingView = () => (
            <div class="flex-grow flex flex-col items-center justify-center text-primary-700 mt-20">
                <div class="animate-spin rounded-full h-16 w-16 border-t-4 border-b-4 border-primary-600 mb-4"></div>
                <h3 class="text-xl font-semibold">Crafting your personalized plan...</h3>
                <p class="text-primary-600/70 mt-2 text-center max-w-md">
                    Analyzing your profile to create the best activities and dietary suggestions for you.
                </p>
            </div>
        );

        const Dashboard = ({ plan, onReset }) => {
            const [msgIndex, setMsgIndex] = useState(0);
            const [fadeKey, setFadeKey] = useState(0); // Used to re-trigger CSS animation

            // Rotate message every 8 seconds
            useEffect(() => {
                if (!plan || !plan.messages || plan.messages.length === 0) return;

                const interval = setInterval(() => {
                    setMsgIndex((prev) => (prev + 1) % plan.messages.length);
                    setFadeKey(prev => prev + 1);
                }, 8000);

                return () => clearInterval(interval);
            }, [plan]);

            if (!plan) return null;

            return (
                <div class="max-w-4xl w-full mx-auto p-4 sm:p-6 lg:p-8 animate-fade-in-up">
                    
                    {/* Top Encouraging Message Banner */}
                    <div class="bg-gradient-to-r from-primary-500 to-teal-500 rounded-2xl shadow-lg p-6 mb-8 text-white relative overflow-hidden">
                        <div class="absolute top-0 right-0 -mt-4 -mr-4 text-white/20">
                            <i class="fa-solid fa-quote-right text-8xl"></i>
                        </div>
                        <div class="relative z-10">
                            <h3 class="text-primary-100 text-sm font-semibold uppercase tracking-wider mb-2">Message of the moment</h3>
                            <p key={fadeKey} class="text-xl sm:text-2xl font-medium italic animate-fade-in-up">
                                "{plan.messages[msgIndex]}"
                            </p>
                            <div class="mt-4 flex gap-1">
                                {plan.messages.map((_, i) => (
                                    <div key={i} class={`h-1.5 rounded-full transition-all duration-300 ${i === msgIndex ? 'w-6 bg-white' : 'w-2 bg-white/30'}`}></div>
                                ))}
                            </div>
                        </div>
                    </div>

                    <div class="grid md:grid-cols-2 gap-6 mb-8">
                        {/* Activities Section */}
                        <div class="bg-white rounded-2xl shadow-md p-6 border border-primary-100">
                            <div class="flex items-center gap-3 mb-6 border-b border-gray-100 pb-4">
                                <div class="bg-orange-100 text-orange-600 p-3 rounded-xl">
                                    <i class="fa-solid fa-person-running text-xl"></i>
                                </div>
                                <h3 class="text-xl font-bold text-gray-800">Recommended Activities</h3>
                            </div>
                            <ul class="space-y-4">
                                {plan.activities.map((activity, i) => (
                                    <li key={i} class="flex items-start gap-3">
                                        <i class="fa-solid fa-check text-primary-500 mt-1"></i>
                                        <span class="text-gray-700 leading-relaxed">{activity}</span>
                                    </li>
                                ))}
                            </ul>
                        </div>

                        {/* Diet Section */}
                        <div class="bg-white rounded-2xl shadow-md p-6 border border-primary-100">
                            <div class="flex items-center gap-3 mb-6 border-b border-gray-100 pb-4">
                                <div class="bg-green-100 text-green-600 p-3 rounded-xl">
                                    <i class="fa-solid fa-apple-whole text-xl"></i>
                                </div>
                                <h3 class="text-xl font-bold text-gray-800">Dietary Suggestions</h3>
                            </div>
                            <ul class="space-y-4">
                                {plan.diet.map((item, i) => (
                                    <li key={i} class="flex items-start gap-3">
                                        <i class="fa-solid fa-check text-primary-500 mt-1"></i>
                                        <span class="text-gray-700 leading-relaxed">{item}</span>
                                    </li>
                                ))}
                            </ul>
                        </div>
                    </div>

                    <div class="text-center">
                        <button 
                            onClick={onReset}
                            class="text-primary-600 hover:text-primary-800 font-medium transition-colors border border-primary-200 bg-white hover:bg-primary-50 px-6 py-2 rounded-lg shadow-sm"
                        >
                            <i class="fa-solid fa-rotate-left mr-2"></i> Update Profile & Get New Plan
                        </button>
                    </div>
                </div>
            );
        };

        const App = () => {
            // State: 'input', 'loading', 'dashboard'
            const [view, setView] = useState('input');
            const [planData, setPlanData] = useState(null);

            const handleFormSubmit = async (formData) => {
                setView('loading');
                const { age, physical, mental } = formData;
                
                // Call API (or fallback)
                const generatedPlan = await generateWellnessPlan(age, physical, mental);
                
                setPlanData(generatedPlan);
                setView('dashboard');
            };

            const handleReset = () => {
                setView('input');
                setPlanData(null);
            };

            return (
                <div class="min-h-screen flex flex-col">
                    {/* Header */}
                    <header class="bg-white shadow-sm sticky top-0 z-50">
                        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8 py-4 flex justify-between items-center">
                            <div class="flex items-center gap-2 text-primary-700">
                                <i class="fa-solid fa-heart-pulse text-2xl"></i>
                                <h1 class="text-xl font-bold tracking-tight">Vitality Coach</h1>
                            </div>
                            {view === 'dashboard' && (
                                <div class="text-sm text-gray-500 font-medium bg-gray-100 px-3 py-1 rounded-full">
                                    Personalized Plan Active
                                </div>
                            )}
                        </div>
                    </header>

                    {/* Main Content Area */}
                    <main class="flex-grow flex flex-col p-4 sm:p-6">
                        {view === 'input' && <InputForm onSubmit={handleFormSubmit} />}
                        {view === 'loading' && <LoadingView />}
                        {view === 'dashboard' && <Dashboard plan={planData} onReset={handleReset} />}
                    </main>

                    {/* Footer */}
                    <footer class="mt-auto py-6 text-center text-gray-400 text-sm">
                        <p>Disclaimer: This app provides general wellness suggestions generated by AI and is not a substitute for professional medical advice.</p>
                    </footer>
                </div>
            );
        };

        const root = ReactDOM.createRoot(document.getElementById('root'));
        root.render(<App />);
    </script>
</body>
</html>
```
