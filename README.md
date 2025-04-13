# OpenAiHeckathonProject

# Atlas AI Voice Assistant
# Created for Hackathon by team anonymous
# A voice-controlled assistant that can open websites, launch applications, and provide time/date information

import speech_recognition as sr
import os
import webbrowser
import datetime
import time
import threading
import platform  # To check operating system


class AtlasAI:
    def __init__(self):
        # Initialize the speech recognizer object
        self.recognizer = sr.Recognizer()
        self.microphone_index = None  # Will be set during runtime
        self.is_listening = False  # Flag to track listening state
        self.debug_mode = True  # Enable detailed logging for hackathon demo

        # Customize speech recognition parameters for better accuracy
        self.recognizer.energy_threshold = 4000  # Higher threshold for noisy environments
        self.recognizer.dynamic_energy_threshold = True  # Dynamically adjust for ambient noise
        self.recognizer.pause_threshold = 0.8  # Shorter pause = more responsive detection

    def speak(self, text):
        """Text-to-speech output with cross-platform support"""
        print(f"Atlas: {text}")  # Print to console for visual feedback

        # Choose text-to-speech method based on operating system
        if platform.system() == 'Darwin':  # macOS
            os.system(f'say "{text}"')
        elif platform.system() == 'Windows':
            # Use Windows built-in TTS (requires Windows)
            try:
                import pyttsx3
                engine = pyttsx3.init()
                engine.say(text)
                engine.runAndWait()
            except ImportError:
                print("Please install pyttsx3 for Windows TTS: pip install pyttsx3")
        else:  # Linux or other
            try:
                os.system(f'espeak "{text}"')  # Requires espeak to be installed
            except:
                pass  # Silently fail if TTS isn't available

    def listen(self):
        """Listen for voice commands with comprehensive error handling"""
        try:
            with sr.Microphone(device_index=self.microphone_index) as source:
                # Visual indicators for user
                print("\nAdjusting for ambient noise...")
                self.is_listening = True

                # Calibrate for background noise - longer duration for better accuracy
                self.recognizer.adjust_for_ambient_noise(source, duration=1.5)

                # Visual feedback to indicate readiness
                print("🎤 Listening... (speak now)")

                try:
                    # Capture audio with optimized timeout parameters
                    audio = self.recognizer.listen(
                        source,
                        timeout=7,  # Increased maximum wait time
                        phrase_time_limit=10  # Longer phrase limit for complex commands
                    )

                    if self.debug_mode:
                        print("Audio captured, processing speech...")

                    # Convert speech to text using Google's API
                    query = self.recognizer.recognize_google(audio).lower()
                    print(f"You said: \"{query}\"")
                    self.is_listening = False
                    return query

                except sr.WaitTimeoutError:
                    # Handle case where no speech is detected
                    print("No speech detected within timeout period")
                    self.speak("I didn't hear anything. Please try again.")
                    self.is_listening = False
                    return None
                except sr.RequestError as e:
                    # Handle network or API errors with specific message
                    print(f"API unavailable: {e}")
                    self.speak("I'm having trouble connecting to the speech service. Check your internet connection.")
                    self.is_listening = False
                    return None
                except sr.UnknownValueError:
                    # Handle unclear speech with constructive feedback
                    print("Speech unrecognizable")
                    self.speak("Sorry, I couldn't understand that. Please speak clearly.")
                    self.is_listening = False
                    return None
                except Exception as e:
                    # Generic error handling with specific error info
                    print(f"Recognition error: {str(e)}")
                    self.speak("I encountered a problem understanding you.")
                    self.is_listening = False
                    return None
        except Exception as e:
            print(f"Microphone error: {str(e)}")
            self.speak("There was a problem with the microphone. Please check your hardware.")
            self.is_listening = False
            return None

    def process_command(self, query):
        """Process user commands with expanded functionality"""
        if not query:
            return

        # Dictionary of websites that can be opened via voice command
        # Expanded for hackathon demonstration
        sites = {
            "youtube": "https://www.youtube.com",
            "wikipedia": "https://www.wikipedia.org",
            "google": "https://google.com",
            "github": "https://github.com",
            "stack overflow": "https://stackoverflow.com",
            "kaggle": "https://kaggle.com",
            "devpost": "https://devpost.com",
            "twitter": "https://twitter.com",
            "linkedin": "https://linkedin.com",
            "facebook": "https://facebook.com",
            "instagram": "https://instagram.com"
        }

        # Check for website opening commands
        for site, url in sites.items():
            if f"open {site}" in query:
                self.speak(f"Opening {site}")
                webbrowser.open(url)
                return

        # Dictionary of applications that can be launched
        # Platform-specific paths for cross-platform compatibility
        if platform.system() == 'Darwin':  # macOS
            apps = {
                "music": "/System/Applications/Music.app",
                "facetime": "/System/Applications/FaceTime.app",
                "calculator": "/System/Applications/Calculator.app",
                "notes": "/System/Applications/Notes.app",
                "calendar": "/System/Applications/Calendar.app",
                "maps": "/System/Applications/Maps.app"
            }
        elif platform.system() == 'Windows':
            apps = {
                "calculator": "calc.exe",
                "notepad": "notepad.exe",
                "paint": "mspaint.exe",
                "word": "winword.exe",
                "excel": "excel.exe"
            }
        else:  # Linux or other
            apps = {
                "calculator": "gnome-calculator",
                "text editor": "gedit",
                "terminal": "gnome-terminal"
            }

        # Check for app launching commands
        for app, path in apps.items():
            if f"open {app}" in query:
                self.speak(f"Launching {app}")
                try:
                    if platform.system() == 'Darwin':
                        os.system(f"open '{path}'")
                    elif platform.system() == 'Windows':
                        os.system(f"start {path}")
                    else:
                        os.system(f"{path} &")
                except Exception as e:
                    print(f"Error launching app: {str(e)}")
                    self.speak(f"I couldn't open {app}")
                return

        # Time query handling with natural language support
        if any(phrase in query for phrase in ["what time", "tell me the time", "current time"]):
            current_time = datetime.datetime.now().strftime("%I:%M %p")
            self.speak(f"The current time is {current_time}")
            return

        # Date query handling with expanded natural language support
        if any(phrase in query for phrase in ["what date", "tell me the date", "what day", "today's date"]):
            current_date = datetime.datetime.now().strftime("%A, %B %d, %Y")
            self.speak(f"Today is {current_date}")
            return

        # System information
        if "system info" in query:
            self.speak(f"You are running {platform.system()} {platform.release()}")
            return

        # Help command
        if any(word in query for word in ["help", "what can you do", "commands"]):
            self.speak(
                "I can open websites like YouTube or Google, launch applications, tell you the time or date, and provide system information. Try saying 'open youtube' or 'what time is it'.")
            return

        # Exit commands - multiple options for flexibility
        if any(word in query for word in ["exit", "quit", "bye", "stop", "shutdown"]):
            self.exit_program()
            return

        # Weather query - placeholder for API integration
        if "weather" in query:
            self.speak("Weather functionality will be added in the next hackathon phase with API integration.")
            return

        # Easter egg for fun demo purposes
        if "hackathon" in query:
            self.speak("Good luck with your hackathon! Remember, the best hacks solve real problems.")
            return

        # Fallback response when no command matches
        self.speak("I'm not sure how to help with that. Try saying 'help' for a list of commands.")

    def test_microphone(self):
        """Test if microphone is capturing audio properly"""
        try:
            self.speak("Testing microphone. Please speak for a few seconds.")
            with sr.Microphone(device_index=self.microphone_index) as source:
                print("Recording test audio...")
                self.recognizer.adjust_for_ambient_noise(source, duration=1)
                audio = self.recognizer.listen(source, timeout=5)
                print("Audio captured. Analyzing...")

                # Try to recognize speech as a simple test
                try:
                    test_text = self.recognizer.recognize_google(audio)
                    self.speak(f"Microphone test successful! I heard: {test_text}")
                    return True
                except sr.UnknownValueError:
                    # Got audio but couldn't understand it
                    self.speak(
                        "Microphone is working, but I couldn't understand what you said. Try again in a quieter environment.")
                    return True
                except Exception as e:
                    print(f"Recognition test failed: {e}")
                    self.speak("I detected audio but couldn't process it.")
                    return False
        except Exception as e:
            print(f"Microphone test failed: {e}")
            self.speak("Microphone test failed. Please check your hardware connections and permissions.")
            return False

    def manual_activate(self):
        """Manual activation"""
        if not self.is_listening:  # Prevent multiple listening sessions
            self.speak("I'm listening")
            # Process in a separate thread to keep UI responsive
            thread = threading.Thread(target=self._listen_thread)
            thread.daemon = True
            thread.start()

    def _listen_thread(self):
        """Thread function for listening to avoid blocking UI"""
        query = self.listen()
        if query:
            self.process_command(query)

    def exit_program(self):
        """Clean exit procedure"""
        self.speak("Goodbye! Shutting down Atlas AI")
        os._exit(0)  # Force immediate exit

    def select_microphone(self):
        """Helper function to select and test microphone"""
        try:
            # List available microphones
            print("\n📋 Available microphones:")
            mics = sr.Microphone.list_microphone_names()

            if not mics:
                print("No microphones detected! Please check your hardware.")
                return False

            for i, name in enumerate(mics):
                print(f"{i}: {name}")

            # Let user select a specific microphone
            if len(mics) > 1:
                try:
                    selection = input("Enter microphone number (or press Enter for default): ")
                    if selection.strip() and selection.isdigit():
                        mic_index = int(selection)
                        if 0 <= mic_index < len(mics):
                            self.microphone_index = mic_index
                            print(f"Selected: {mics[mic_index]}")
                        else:
                            print("Invalid selection, using default microphone")
                except Exception as e:
                    print(f"Error in selection: {e}, using default microphone")

            print(
                f"Using microphone: {mics[self.microphone_index] if self.microphone_index is not None else 'Default'}")
            return True
        except Exception as e:
            print(f"Error listing microphones: {str(e)}")
            print("Continuing with default microphone if available")
            return False

    def run(self):
        """Main execution loop with improved structure"""
        print("=" * 50)
        print(" ATLAS AI - Voice Assistant Hackathon Project")
        print("=" * 50)
        print("Voice Commands Available - say commands like:")
        print(" 'Open YouTube'")
        print(" 'What time is it?'")
        print(" 'What date is today?'")
        print(" 'System info'")
        print(" 'Help'")
        print(" 'Exit'")
        print("=" * 50)

        # Select and test microphone on startup
        if not self.select_microphone():
            print("WARNING: Microphone setup issue detected.")

        # Test the microphone
        mic_working = self.test_microphone()

        if not mic_working:
            print("\n⚠️ Microphone test failed! Voice recognition may not work properly.")
            print("Troubleshooting tips:")
            print("1. Check if your microphone is connected properly")
            print("2. Ensure you've granted microphone permissions to Python")
            print("3. Try selecting a different microphone")
            print("4. Install PyAudio if it's missing: pip install pyaudio\n")

        self.speak("Atlas AI initialized. Say a command to begin.")

        # Start listening immediately
        self.manual_activate()

        # Main loop with better error handling
        while True:
            try:
                # Use a small delay to prevent high CPU usage
                time.sleep(0.1)

                # In this version without keyboard, we'll need to continuously listen
                if not self.is_listening:
                    self.manual_activate()

            except KeyboardInterrupt:
                self.exit_program()
            except Exception as e:
                print(f"Unexpected error in main loop: {str(e)}")
                time.sleep(1)  # Prevent error spam


if __name__ == '__main__':
    # Check for required packages
    missing_packages = []
    try:
        import speech_recognition as sr
    except ImportError:
        missing_packages.append("SpeechRecognition")
    try:
        sr.Microphone()
    except (ImportError, AttributeError):
        missing_packages.append("PyAudio")

    # If packages are missing, provide installation instructions
    if missing_packages:
        print("=" * 50)
        print("Missing required packages. Please install:")
        print(f"pip install {' '.join(missing_packages)}")
        print("=" * 50)
        input("Press Enter to exit...")
        exit()

    try:
        print("Starting Atlas AI...")
        assistant = AtlasAI()
        assistant.run()  # Start the assistant
    except Exception as e:
        print(f"Fatal error: {str(e)}")
        print("Atlas AI could not start.")
        input("Press Enter to exit...")



      #tool required
      #install pycharm community version or profesional version
      #install jet brain tool
      




        #installation must be required for this 
        #pip install SpeechRecognition
        #pip install wikipedia
        #pip install pyaudio
        #pip install soundevice
        #pip install numphy
        #pip install scipy
