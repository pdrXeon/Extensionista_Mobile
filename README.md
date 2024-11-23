import React, { useState } from 'react';
import { View, Text, TextInput, TouchableOpacity, Button, StyleSheet, ImageBackground, ScrollView } from 'react-native';
import AsyncStorage from '@react-native-async-storage/async-storage';
import { NavigationContainer } from '@react-navigation/native';
import { createStackNavigator } from '@react-navigation/stack';
import { useEffect } from 'react';

// Função para salvar dados no AsyncStorage
const saveCredentials = async (name, email, password) => {
  try {
    await AsyncStorage.setItem('user', JSON.stringify({ name, email, password }));
  } catch (error) {
    console.log(error);
  }
};

// Função para obter dados do AsyncStorage
const getCredentials = async () => {
  try {
    const user = await AsyncStorage.getItem('user');
    return user ? JSON.parse(user) : null;
  } catch (error) {
    console.log(error);
    return null;
  }
};

// Tela de Login e Cadastro
function LoginScreen({ navigation }) {
  const [isLogin, setIsLogin] = useState(true);
  const [name, setName] = useState(''); 
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  const [confirmPassword, setConfirmPassword] = useState('');

  const handleLogin = async () => {
    const savedCredentials = await getCredentials();

    if (savedCredentials && savedCredentials.email === email && savedCredentials.password === password) {
      navigation.navigate('Home', { name: savedCredentials.name, email });
    } else {
      alert('Email ou senha incorretos');
    }
  };

  const handleSignUp = async () => {
    if (password !== confirmPassword) {
      alert('As senhas não coincidem');
      return;
    }

    await saveCredentials(name, email, password);
    alert('Conta criada com sucesso!');
    setIsLogin(true);
  };

  return (
    <ImageBackground
      source={require('./assets/fundo.png')} // Coloque a sua imagem aqui
      style={styles.background}
    >
      <View style={styles.container}>
        <Text style={styles.title}>{isLogin ? 'Login' : 'Cadastro'}</Text>

        {!isLogin && (
          <TextInput
            style={styles.input}
            placeholder="Nome"
            value={name}
            onChangeText={setName}
          />
        )}

        <TextInput
          style={styles.input}
          placeholder="Email"
          value={email}
          onChangeText={setEmail}
          keyboardType="email-address"
          autoCapitalize="none"
        />

        <TextInput
          style={styles.input}
          placeholder="Senha"
          value={password}
          onChangeText={setPassword}
          secureTextEntry
        />

        {!isLogin && (
          <TextInput
            style={styles.input}
            placeholder="Confirme a senha"
            value={confirmPassword}
            onChangeText={setConfirmPassword}
            secureTextEntry
          />
        )}

        <Button
          title={isLogin ? 'Entrar' : 'Cadastrar'}
          onPress={isLogin ? handleLogin : handleSignUp}
        />

        <TouchableOpacity onPress={() => setIsLogin(!isLogin)}>
          <Text style={styles.switchText}>
            {isLogin ? 'Ainda não tem uma conta? Cadastre-se' : 'Já tem uma conta? Faça Login'}
          </Text>
        </TouchableOpacity>
      </View>
    </ImageBackground>
  );
}

// Tela inicial após o login
function HomeScreen({ route, navigation }) {
  const { name, email } = route.params;
  const [cards, setCards] = useState([
    { id: 1, title: 'Sítio JONOSAKE 🍖🍗🐟', description: 'Dia 15 novembro 2024 - Sexta-Feira\n\n💰 VALORES:R$ 210,00\n\n⏰Saida:06:30hs\n⏰Retorno:17:00hs\n\n👉 PACOTE INCLUI:\n▶Transporte de turismo\n▶Café da manhãm, Almoço\n▶Bebida liberada(11:00 as 15:00)' },
    { id: 2, title: '🇩🇪BAUERNFEST🇩🇪 ', description: 'Dia 06 julho 2025 - Sábado\n\n💰 VALORES:R$ 100,00\n\n⏰Saida:09:00hs\n⏰Retorno:21:00hs\n\n🚨INCLUSO:\n▶Transporte🚍\n▶Kit lanche na ida.\n▶Seguro Viagem\n▶Serviço de Bordo ' },
    { id: 3, title: '🗺️Nova Friburgo🗺️', description: 'Dia 20 dezembro 2024 - Sexta-Feira\n\n💰 VALORES:R$ 180,00\n\n⏰Saida:07:30hs\n⏰Retorno:18:00hs\n\n👉 PACOTE INCLUI:\n▶Transporte de turismo\n▶Serviço de bordo\n▶Almoço no local' },
    { id: 4, title: '💦Aldeia das Aguas🏖️', description: 'Dia 25 Janeiro 2025 - Sábado\n\n💰 VALORES:R$ 150,00\n\n⏰Saida:07:00hs\n⏰Retorno:16:00hs\n\n👉 PACOTE INCLUI:\n▶Transporte de turismo\n▶Serviço de bordo\n▶Almoço no local' },
  ]);
  // Estado para os passeios favoritos
const [favoriteTrips, setFavoriteTrips] = useState([]);

// Função para salvar passeios favoritos no AsyncStorage
const saveFavoriteTrips = async (trips) => {
  try {
    await AsyncStorage.setItem('favoriteTrips', JSON.stringify(trips));
  } catch (error) {
    console.log(error);
  }
};

// Função para recuperar passeios favoritos do AsyncStorage
const getFavoriteTrips = async () => {
  try {
    const savedTrips = await AsyncStorage.getItem('favoriteTrips');
    return savedTrips ? JSON.parse(savedTrips) : [];
  } catch (error) {
    console.log(error);
    return [];
  }
};

// Recupera os passeios salvos ao carregar a tela
useEffect(() => {
  const loadFavoriteTrips = async () => {
    const trips = await getFavoriteTrips();
    setFavoriteTrips(trips);
  };
  loadFavoriteTrips();
}, []);

const handleAddToFavorites = async (card) => {
  const updatedFavorites = [...favoriteTrips, card];
  setFavoriteTrips(updatedFavorites);
  await saveFavoriteTrips(updatedFavorites);
  alert(`${card.title} foi adicionado a suas viagens`);
};

  const handleCardPress = (card) => {
    navigation.navigate('CardDetails', { card });
  };

  const handleLogout = async () => {
    await AsyncStorage.removeItem('user');
    navigation.navigate('Login');
  };
  const handleViewFavorites = () => {
    if (favoriteTrips.length === 0) {
      alert('Você ainda não salvou nenhum passeio.');
    } else {
      const tripsList = favoriteTrips.map((trip) => `${trip.title}: ${trip.description}`).join('\n\n');
      alert(`Suas Viagens:\n\n${tripsList}`);
    }
  };
  const handleNewButtonPress = () => {
    alert('🚍 EMBARQUES:\n> Rua Francisco Mota\n> Rodoviária de Campo Grande\n> Demais embarque a combinar ');
  };
  const handleClearFavorites = async () => {
    setFavoriteTrips([]); // Limpa o estado local
    await saveFavoriteTrips([]); // Limpa os favoritos no AsyncStorage
    alert('Todos os passeios foram removidos das suas viagens!');
  };

  return (
    <ImageBackground
      source={require('./assets/tela2.jpg')}
      style={styles.homeBackground}
    >
      <View style={styles.homeContainer}>
        <Text style={styles.homeText}>Bem-vindo(a), {name}!</Text>

        <View style={styles.buttonContainer}>
          <Button title="MInhas Viagens" onPress={handleViewFavorites} color="#072aed" />
          <Button title="Embarques" onPress={handleNewButtonPress} color="#FFA500" />
          <Button title="Limpar" onPress={handleClearFavorites} color="#ec5353" />
          <Button title="Sair" onPress={handleLogout} color="#ff0000" />
        </View>

        <ScrollView contentContainerStyle={styles.cardContainer}>
  {cards.map((card) => (
    <View key={card.id} style={styles.card}>
      <View style={styles.cardContent}>
        <View>
          <Text style={styles.cardTitle}>{card.title}</Text>
          <Text style={styles.cardDescription}>{card.description}</Text>
        </View>
        <TouchableOpacity 
          style={styles.cardButton} 
          onPress={() => handleAddToFavorites(card)}  // Modificação aqui
        >
          <Text style={styles.buttonText}>Selecionar</Text>
        </TouchableOpacity>
      </View>
    </View>
  ))}
</ScrollView>
      </View>
    </ImageBackground>
  );  
}
const Stack = createStackNavigator();

export default function App() {
  return (
    <NavigationContainer>
      <Stack.Navigator>
        <Stack.Screen name="Login" component={LoginScreen} options={{ headerShown: false }} />
        <Stack.Screen name="Home" component={HomeScreen} options={{ headerShown: false }} />
        
      </Stack.Navigator>
    </NavigationContainer>
  );
}

const styles = StyleSheet.create({
  background: {
    flex: 1,
    resizeMode: 'cover', // Ajusta a imagem para cobrir a tela inteira
  },
  homeBackground: {
    flex: 1,
    resizeMode: 'cover', // Ajusta a imagem para cobrir a tela inteira
  },
  container: {
    flex: 1,
    justifyContent: 'center',
    padding: 20,
  },
  title: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
    textAlign: 'center',
  },
  input: {
    height: 50,
    borderColor: '#ccc',
    borderWidth: 1,
    borderRadius: 5,
    paddingHorizontal: 10,
    marginBottom: 15,
    backgroundColor: '#fff',
  },
  switchText: {
    marginTop: 15,
    textAlign: 'center',
    color: '#fff',
  },
  homeContainer: {
    flex: 1,
    padding: 20,
    backgroundColor: 'rgba(255, 255, 255, 0.3)', // Fundo branco semitransparente para o conteúdo
  },
  homeText: {
    fontSize: 24,
    fontWeight: 'bold',
    marginBottom: 20,
  },
  buttonContainer: {
    flexDirection: 'row',
    justifyContent: 'space-between',
    marginBottom: 20,
  },
  cardContainer: {
    paddingBottom: 20,
  },
  card: {
    backgroundColor: '#fff',
    padding: 20,
    borderRadius: 10,
    marginBottom: 15,
    shadowColor: '#000',
    shadowOffset: { width: 0, height: 2 },
    shadowOpacity: 0.3,
    shadowRadius: 3,
    elevation: 5,
  },
  cardTitle: {
    fontSize: 18,
    fontWeight: 'bold',
  },
  cardDescription: {
    fontSize: 14,
    color: '#666',
    marginTop: 5,
  },
  detailContainer: {
    flex: 1,
    justifyContent: 'top',
    alignItems: 'center',
    padding: 20,
  },
  cardContent: {
    flexDirection: 'row',
    alignItems: 'flex-start',
    justifyContent: 'space-between',
  },
  cardButton: {
    backgroundColor: '#4CAF50',
    paddingVertical: 10,
    paddingHorizontal: 15,
    borderRadius: 5,
  },
  buttonText: {
    color: '#fff',
    fontWeight: 'bold',
  },
});

