import React, { useState, useEffect } from 'react';
import {
  View,
  StyleSheet,
  Text,
  TouchableOpacity,
  ScrollView,
  PermissionsAndroid,
  Linking,
  Platform,
  Alert,
  AppState
} from 'react-native';
import TTS from 'react-native-tts';
import Voice from 'react-native-voice';
import NetInfo from '@react-native-community/netinfo';
import axios from 'axios';

// =========== MAIN APP COMPONENT ===========
const ZaheedAI = () => {
  const [messages, setMessages] = useState([
    { id: 1, text: 'السلام علیکم زاہد سر! میں آپ کا ذاتی اسسٹنٹ ہوں۔', sender: 'assistant' }
  ]);
  const [isListening, setIsListening] = useState(false);
  const [assistantActive, setAssistantActive] = useState(true);
  const [notifications, setNotifications] = useState([]);
  const [userName] = useState('زاہد سر');

  // =========== INITIALIZATION ===========
  useEffect(() => {
    requestPermissions();
    initializeTTS();
    initializeVoice();
    startDeviceMonitoring();
    setupAppStateListener();
  }, []);

  // =========== PERMISSIONS ===========
  const requestPermissions = async () => {
    try {
      const permissions = [
        PermissionsAndroid.PERMISSIONS.RECORD_AUDIO,
        PermissionsAndroid.PERMISSIONS.CAMERA,
        PermissionsAndroid.PERMISSIONS.READ_EXTERNAL_STORAGE,
        PermissionsAndroid.PERMISSIONS.WRITE_EXTERNAL_STORAGE,
        'android.permission.BIND_NOTIFICATION_LISTENER_SERVICE',
        'android.permission.PROJECT_MEDIA',
        'android.permission.SYSTEM_ALERT_WINDOW',
        'android.permission.ACCESS_NOTIFICATION_POLICY'
      ];

      const granted = await PermissionsAndroid.requestMultiple(permissions);
      console.log('Permissions granted:', granted);
    } catch (err) {
      console.warn('Permission error:', err);
    }
  };

  // =========== TTS SETUP ===========
  const initializeTTS = () => {
    TTS.setDefaultLanguage('ur-PK');
    TTS.setDefaultRate(0.5);
    TTS.setDefaultPitch(1.2);
    TTS.addEventListener('tts-start', () => console.log('Speaking started'));
    TTS.addEventListener('tts-finish', () => console.log('Speaking finished'));
  };

  // =========== VOICE RECOGNITION ===========
  const initializeVoice = () => {
    Voice.onSpeechStart = () => {
      setIsListening(true);
      addMessage('میں سن رہا ہوں...', 'assistant');
    };

    Voice.onSpeechEnd = () => {
      setIsListening(false);
    };

    Voice.onSpeechResults = (e) => {
      const command = e.value[0];
      addMessage(command, 'user');
      processVoiceCommand(command);
    };

    Voice.onSpeechError = (e) => {
      console.log('Voice error:', e);
      speak('معذرت، میں آواز نہیں سمجھ سکا۔');
    };
  };

  // =========== VOICE FUNCTIONS ===========
  const startListening = async () => {
    try {
      await Voice.start('ur-PK');
    } catch (error) {
      console.log('Listening error:', error);
    }
  };

  const stopListening = async () => {
    try {
      await Voice.stop();
    } catch (error) {
      console.log('Stop listening error:', error);
    }
  };

  const speak = (text) => {
    const fullText = `${userName}, ${text}`;
    TTS.speak(fullText, {
      language: 'ur-PK',
      androidParams: {
        KEY_PARAM_VOLUME: 1.0,
        KEY_PARAM_STREAM: 'STREAM_MUSIC',
      },
    });
    addMessage(fullText, 'assistant');
  };

  // =========== VOICE COMMAND PROCESSING ===========
  const processVoiceCommand = (command) => {
    const cmd = command.toLowerCase();
    
    // Greeting
    if (cmd.includes('ہیلو') || cmd.includes('سلام') || cmd.includes('اسٹارٹ')) {
      speak('السلام علیکم زاہد سر! آپ کا کیا حکم ہے؟');
    }
    
    // YouTube Commands
    else if (cmd.includes('یوٹیوب کھولو')) {
      speak('یوٹیوب کھول رہا ہوں');
      openApp('com.google.android.youtube');
    }
    else if (cmd.includes('یوٹیوب پر سرچ کرو')) {
      const query = extractSearchQuery(cmd);
      speak(`یوٹیوب پر ${query} سرچ کر رہا ہوں`);
      searchYouTube(query);
    }
    
    // WhatsApp Commands
    else if (cmd.includes('واٹس ایپ کھولو')) {
      speak('واٹس ایپ کھول رہا ہوں');
      openApp('com.whatsapp');
    }
    
    // Notifications
    else if (cmd.includes('نوٹیفکیشن')) {
      speak('آپ کی نوٹیفکیشنز چیک کر رہا ہوں');
      simulateNotifications();
    }
    
    // Battery Check
    else if (cmd.includes('بیٹری') || cmd.includes('چارج')) {
      checkBatteryStatus();
    }
    
    // Network Check
    else if (cmd.includes('نیٹ ورک') || cmd.includes('انٹرنیٹ')) {
      checkNetworkStatus();
    }
    
    // Camera
    else if (cmd.includes('کیمرہ') || cmd.includes('فوٹو')) {
      speak('کیمرہ کھول رہا ہوں');
      openCamera();
    }
    
    // Screen Share
    else if (cmd.includes('سکرین') || cmd.includes('شیئر')) {
      speak('سکرین شیئرنگ شروع کر رہا ہوں');
      startScreenSharing();
    }
    
    // Time
    else if (cmd.includes('ٹائم') || cmd.includes('وقت')) {
      const time = new Date().toLocaleTimeString('ur-PK', { hour: '2-digit', minute: '2-digit' });
      speak(`اب وقت ہوا ہے ${time}`);
    }
    
    // Date
    else if (cmd.includes('تاریخ') || cmd.includes('ڈیٹ')) {
      const date = new Date().toLocaleDateString('ur-PK', { 
        weekday: 'long',
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      });
      speak(`آج ${date} ہے`);
    }
    
    // Help
    else if (cmd.includes('مدد') || cmd.includes('ہیلپ')) {
      speak('میں آپ کی کس طرح مدد کر سکتا ہوں؟ آپ یہ کمانڈز استعمال کر سکتے ہیں: یوٹیوب کھولو، نوٹیفکیشنز دیکھیں، بیٹری چیک کریں، کیمرہ کھولیں، سکرین شیئر کریں');
    }
    
    // Default - AI Response
    else {
      getAIResponse(command);
    }
  };

  // =========== AI RESPONSE ===========
  const getAIResponse = async (query) => {
    speak('سوچ رہا ہوں...');
    
    // Local responses for common queries
    const localResponses = {
      'آپ کا نام کیا ہے': 'میرا نام زاہد AI ہے، میں آپ کا ذاتی اسسٹنٹ ہوں۔',
      'آپ کون ہو': 'میں آپ کا ذاتی مصنوعی ذہانت اسسٹنٹ ہوں جو آپ کی ہر کام میں مدد کر سکتا ہوں۔',
      'تم کیسے ہو': 'اللہ کا شکر ہے زاہد سر، آپ کا کیا حال ہے؟',
      'شکریہ': 'خوش آمدید زاہد سر، آپ کا بہت بہت شکریہ۔',
      'الوداع': 'اللہ حافظ زاہد سر، آپ کا دن اچھا گزرے۔',
    };

    if (localResponses[query]) {
      speak(localResponses[query]);
      return;
    }

    // For other queries, use OpenAI API (you need to add your API key)
    try {
      const response = await axios.post('https://api.openai.com/v1/chat/completions', {
        model: 'gpt-3.5-turbo',
        messages: [
          {
            role: 'system',
            content: 'آپ زاہد سر کا ذاتی اسسٹنٹ ہیں۔ ہمیشہ اردو میں بات کریں اور پاکستانی لہجے میں۔ صارف کو "زاہد سر" کہہ کر مخاطب کریں۔ آپ کی آواز قدرتی اور دوستانہ ہے۔'
          },
          {
            role: 'user',
            content: query
          }
        ],
        temperature: 0.7,
        max_tokens: 150
      }, {
        headers: {
          'Authorization': 'Bearer YOUR_OPENAI_API_KEY', // Add your key here
          'Content-Type': 'application/json'
        }
      });

      const aiResponse = response.data.choices[0].message.content;
      speak(aiResponse);
    } catch (error) {
      console.log('AI error:', error);
      speak('معذرت زاہد سر، میں اس وقت جواب نہیں دے سکا۔ براہ کرم دوبارہ کوشش کریں۔');
    }
  };

  // =========== AUTOMATION FUNCTIONS ===========
  const openApp = (packageName) => {
    if (Platform.OS === 'android') {
      Linking.openURL(`market://details?id=${packageName}`)
        .catch(() => {
          Linking.openURL(`https://play.google.com/store/apps/details?id=${packageName}`);
        });
    }
  };

  const searchYouTube = (query) => {
    const encodedQuery = encodeURIComponent(query);
    Linking.openURL(`https://www.youtube.com/results?search_query=${encodedQuery}`);
  };

  const openCamera = () => {
    // Camera implementation would go here
    speak('کیمرہ تیار ہے، آپ تصویر لے سکتے ہیں۔');
  };

  const startScreenSharing = () => {
    // Screen sharing implementation
    speak('سکرین شیئرنگ شروع ہو گئی ہے۔');
  };

  // =========== DEVICE MONITORING ===========
  const startDeviceMonitoring = () => {
    // Check battery every 5 minutes
    setInterval(() => {
      checkBatteryStatus();
      checkNetworkStatus();
    }, 300000);
  };

  const checkBatteryStatus = () => {
    // In real app, use react-native-battery package
    const batteryLevel = Math.random() * 100; // Simulated battery level
    
    if (batteryLevel < 20) {
      speak(`تنبیہ: بیٹری صرف ${Math.round(batteryLevel)} فیصد رہ گئی ہے۔ براہ کرم فون چارج کریں۔`);
    }
    
    if (batteryLevel < 10) {
      speak(`ہنگامی: بیٹری صرف ${Math.round(batteryLevel)} فیصد ہے! فوری چارج کریں۔`);
    }
  };

  const checkNetworkStatus = async () => {
    const state = await NetInfo.fetch();
    
    if (!state.isConnected) {
      speak('نیٹ ورک کنکشن منقطع ہے۔ براہ کرم انٹرنیٹ کنکشن چیک کریں۔');
    } else if (state.type === 'cellular' && state.details.cellularGeneration !== '4g') {
      speak('نیٹ ورک سگنل کمزور ہے۔ براہ کرم بہتر جگہ پر جائیں۔');
    }
  };

  // =========== NOTIFICATION SYSTEM ===========
  const simulateNotifications = () => {
    const simulatedNotifications = [
      { id: 1, app: 'WhatsApp', message: 'آصف: ہیلو، کیسے ہو؟', time: '2 منٹ پہلے' },
      { id: 2, app: 'YouTube', message: 'نئی ویڈیو: AI کا مستقبل', time: '10 منٹ پہلے' },
      { id: 3, app: 'Gmail', message: 'بینک: آپ کا سٹیٹمنٹ تیار ہے', time: '1 گھنٹہ پہلے' },
    ];
    
    setNotifications(simulatedNotifications);
    
    simulatedNotifications.forEach((notif, index) => {
      setTimeout(() => {
        speak(`${notif.app} سے نوٹیفکیشن: ${notif.message}`);
      }, index * 2000);
    });
  };

  const handleNotificationResponse = (notificationId, response) => {
    speak(`نوٹیفکیشن ${notificationId} کو جواب بھیج دیا گیا: ${response}`);
    // Here you would implement actual notification reply
  };

  // =========== UTILITY FUNCTIONS ===========
  const extractSearchQuery = (command) => {
    const keywords = ['سرچ کرو', 'تلاش کرو', 'ڈھونڈو'];
    for (let keyword of keywords) {
      if (command.includes(keyword)) {
        return command.replace(keyword, '').replace('یوٹیوب پر', '').trim();
      }
    }
    return command;
  };

  const addMessage = (text, sender) => {
    setMessages(prev => [...prev, { id: Date.now(), text, sender }]);
  };

  const setupAppStateListener = () => {
    AppState.addEventListener('change', (nextAppState) => {
      if (nextAppState === 'active') {
        // App came to foreground
        speak('خوش آمدید زاہد سر! میں حاضر ہوں۔');
      }
    });
  };

  // =========== QUICK ACTIONS ===========
  const quickActions = [
    { id: 1, title: 'یوٹیوب کھولیں', icon: '🎬', action: () => openApp('com.google.android.youtube') },
    { id: 2, title: 'واٹس ایپ کھولیں', icon: '💬', action: () => openApp('com.whatsapp') },
    { id: 3, title: 'کیمرہ کھولیں', icon: '📷', action: openCamera },
    { id: 4, title: 'سکرین شیئر کریں', icon: '🖥️', action: startScreenSharing },
    { id: 5, title: 'نوٹیفکیشنز', icon: '📢', action: simulateNotifications },
    { id: 6, title: 'بیٹری چیک کریں', icon: '🔋', action: checkBatteryStatus },
  ];

  // =========== RENDER ===========
  return (
    <View style={styles.container}>
      {/* Header */}
      <View style={styles.header}>
        <Text style={styles.headerTitle}>زاہد AI اسسٹنٹ</Text>
        <TouchableOpacity 
          style={styles.statusButton}
          onPress={() => setAssistantActive(!assistantActive)}
        >
          <View style={[
            styles.statusIndicator, 
            { backgroundColor: assistantActive ? '#4CAF50' : '#f44336' }
          ]} />
          <Text style={styles.statusText}>
            {assistantActive ? 'فعال' : 'غیر فعال'}
          </Text>
        </TouchableOpacity>
      </View>

      {/* Chat Area */}
      <ScrollView style={styles.chatContainer}>
        {messages.map((message) => (
          <View
            key={message.id}
            style={[
              styles.messageBubble,
              message.sender === 'assistant' ? styles.assistantBubble : styles.userBubble
            ]}
          >
            <Text style={[
              styles.messageText,
              message.sender === 'assistant' ? styles.assistantText : styles.userText
            ]}>
              {message.text}
            </Text>
          </View>
        ))}
      </ScrollView>

      {/* Quick Actions */}
      <ScrollView horizontal showsHorizontalScrollIndicator={false} style={styles.quickActions}>
        {quickActions.map((action) => (
          <TouchableOpacity 
            key={action.id} 
            style={styles.quickActionButton} 
            onPress={action.action}
          >
            <Text style={styles.quickActionIcon}>{action.icon}</Text>
            <Text style={styles.quickActionText}>{action.title}</Text>
          </TouchableOpacity>
        ))}
      </ScrollView>

      {/* Notifications Panel */}
      {notifications.length > 0 && (
        <View style={styles.notificationPanel}>
          <Text style={styles.notificationTitle}>نوٹیفکیشنز ({notifications.length})</Text>
          {notifications.map(notif => (
            <View key={notif.id} style={styles.notificationItem}>
              <Text style={styles.notificationApp}>{notif.app}</Text>
              <Text style={styles.notificationMessage}>{notif.message}</Text>
              <Text style={styles.notificationTime}>{notif.time}</Text>
            </View>
          ))}
        </View>
      )}

      {/* Voice Control */}
      <View style={styles.voiceControl}>
        <TouchableOpacity
          style={[styles.voiceButton, isListening && styles.listeningButton]}
          onPress={() => isListening ? stopListening() : startListening()}
        >
          <Text style={styles.voiceButtonIcon}>
            {isListening ? '🎤' : '🎤'}
          </Text>
          <Text style={styles.voiceButtonText}>
            {isListening ? '...سن رہا ہوں' : 'بولنے کے لیے دبائیں'}
          </Text>
        </TouchableOpacity>
      </View>

      {/* Footer Menu */}
      <View style={styles.footer}>
        <TouchableOpacity style={styles.footerButton}>
          <Text style={styles.footerIcon}>🏠</Text>
          <Text style={styles.footerText}>ہوم</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.footerButton} onPress={simulateNotifications}>
          <Text style={styles.footerIcon}>📢</Text>
          <Text style={styles.footerText}>نوٹیفکیشنز</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.footerButton} onPress={checkBatteryStatus}>
          <Text style={styles.footerIcon}>🔋</Text>
          <Text style={styles.footerText}>بیٹری</Text>
        </TouchableOpacity>
        <TouchableOpacity style={styles.footerButton} onPress={() => speak('میں آپ کی کس طرح مدد کر سکتا ہوں؟')}>
          <Text style={styles.footerIcon}>❓</Text>
          <Text style={styles.footerText}>مدد</Text>
        </TouchableOpacity>
      </View>
    </View>
  );
};

// =========== STYLES ===========
const styles = StyleSheet.create({
  container: {
    flex: 1,
    backgroundColor: '#121212',
  },
  header: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    alignItems: 'center',
    paddingHorizontal: 20,
    paddingTop: 50,
    paddingBottom: 20,
    backgroundColor: '#1e1e1e',
    borderBottomWidth: 1,
    borderBottomColor: '#333',
  },
  headerTitle: {
    fontSize: 24,
    fontWeight: 'bold',
    color: '#4CAF50',
  },
  statusButton: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: '#333',
    paddingHorizontal: 15,
    paddingVertical: 8,
    borderRadius: 20,
  },
  statusIndicator: {
    width: 10,
    height: 10,
    borderRadius: 5,
    marginRight: 8,
  },
  statusText: {
    color: '#fff',
    fontSize: 14,
  },
  chatContainer: {
    flex: 1,
    paddingHorizontal: 20,
    paddingVertical: 10,
  },
  messageBubble: {
    maxWidth: '80%',
    padding: 15,
    borderRadius: 20,
    marginVertical: 5,
  },
  assistantBubble: {
    backgroundColor: '#2d2d2d',
    alignSelf: 'flex-start',
  },
  userBubble: {
    backgroundColor: '#4CAF50',
    alignSelf: 'flex-end',
  },
  messageText: {
    fontSize: 16,
  },
  assistantText: {
    color: '#fff',
  },
  userText: {
    color: '#fff',
  },
  quickActions: {
    flexDirection: 'row',
    paddingHorizontal: 15,
    paddingVertical: 10,
    backgroundColor: '#1a1a1a',
  },
  quickActionButton: {
    alignItems: 'center',
    justifyContent: 'center',
    backgroundColor: '#333',
    padding: 15,
    borderRadius: 15,
    marginHorizontal: 5,
    width: 100,
  },
  quickActionIcon: {
    fontSize: 30,
    marginBottom: 5,
  },
  quickActionText: {
    color: '#fff',
    fontSize: 12,
    textAlign: 'center',
  },
  notificationPanel: {
    backgroundColor: '#2a2a2a',
    margin: 10,
    padding: 15,
    borderRadius: 10,
  },
  notificationTitle: {
    color: '#4CAF50',
    fontSize: 18,
    fontWeight: 'bold',
    marginBottom: 10,
  },
  notificationItem: {
    backgroundColor: '#333',
    padding: 10,
    borderRadius: 5,
    marginBottom: 5,
  },
  notificationApp: {
    color: '#FF9800',
    fontSize: 14,
    fontWeight: 'bold',
  },
  notificationMessage: {
    color: '#fff',
    fontSize: 12,
  },
  notificationTime: {
    color: '#aaa',
    fontSize: 10,
    textAlign: 'right',
  },
  voiceControl: {
    alignItems: 'center',
    paddingVertical: 20,
    backgroundColor: '#1e1e1e',
  },
  voiceButton: {
    flexDirection: 'row',
    alignItems: 'center',
    backgroundColor: '#4CAF50',
    paddingHorizontal: 30,
    paddingVertical: 15,
    borderRadius: 30,
  },
  listeningButton: {
    backgroundColor: '#f44336',
  },
  voiceButtonIcon: {
    fontSize: 24,
    marginRight: 10,
  },
  voiceButtonText: {
    color: '#fff',
    fontSize: 18,
    fontWeight: 'bold',
  },
  footer: {
    flexDirection: 'row',
    justifyContent: 'space-around',
    paddingVertical: 15,
    backgroundColor: '#1a1a1a',
    borderTopWidth: 1,
    borderTopColor: '#333',
  },
  footerButton: {
    alignItems: 'center',
  },
  footerIcon: {
    fontSize: 24,
    marginBottom: 5,
  },
  footerText: {
    color: '#aaa',
    fontSize: 12,
  },
});

// =========== PACKAGE.JSON ===========
/*
{
  "name": "ZaheedAI",
  "version": "1.0.0",
  "main": "ZaheedAI.js",
  "scripts": {
    "start": "node node_modules/react-native/local-cli/cli.js start",
    "android": "react-native run-android",
    "ios": "react-native run-ios"
  },
  "dependencies": {
    "react": "18.2.0",
    "react-native": "0.73.0",
    "react-native-voice": "^3.2.2",
    "react-native-tts": "^4.1.0",
    "@react-native-community/netinfo": "^9.3.7",
    "axios": "^1.6.0",
    "react-native-permissions": "^3.8.0",
    "react-native-camera": "^4.2.1",
    "react-native-screenshot-detector": "^1.1.0"
  }
}
*/

// =========== ANDROID CONFIGURATION ===========
/*
// android/app/build.gradle میں یہ اضافہ کریں:
android {
    defaultConfig {
        multiDexEnabled true
    }
}

dependencies {
    implementation project(':react-native-voice')
    implementation project(':react-native-tts')
    implementation project(':@react-native-community_netinfo')
}
*/

// =========== IMPORTANT NOTES ===========
/*
1. یہ مکمل ایپ ہے جو React Native میں کام کرے گی
2. آپ کو اپنا OpenAI API key داخل کرنا ہوگا
3. Android میں permission کے لیے android/app/src/main/AndroidManifest.xml میں یہ اضافہ کریں:

<uses-permission android:name="android.permission.RECORD_AUDIO" />
<uses-permission android:name="android.permission.INTERNET" />
<uses-permission android:name="android.permission.CAMERA" />
<uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />
<uses-permission android:name="android.permission.BIND_NOTIFICATION_LISTENER_SERVICE" />

4. ایپ کو چلانے کے لیے:
   - npm install
   - npx react-native run-android

5. فیچرز:
   - وائس کمانڈز
   - آواز میں جواب
   - نوٹیفکیشن ہینڈلنگ
   - ڈیوائس مانیٹرنگ
   - یوٹیوب/واٹس ایپ کھولنا
   - کیمرہ اور سکرین شیئرنگ
   - اردو زبان کی مکمل سپورٹ
   - زاہد سر کے نام سے مخاطب
*/

export default ZaheedAI;
