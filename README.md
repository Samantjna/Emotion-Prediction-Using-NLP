# Emotion-Prediction-Using-NLP (ENG) <img src="https://media.giphy.com/media/3o7WIx7urV838kHFzW/giphy.gif" width="50">

<br>

## Introduction
This project deals with the recognition of text emotions, which can be used in the analysis of social media posts, customer reviews or conversations.
     The goal is to create a model that identifies and classifies different emotions in a text, such as joy, anger & fear.

### - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

### Technology used:
Python ✦ TensorFlow/Keras ✦ NLTK ✦ scikit-learn ✦ Pandas

### - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

## Work stages
<b> ✦ Data collection: </b> Using dataset such as the <a href=https://www.kaggle.com/datasets/abdallahwagih/emotion-dataset/data>"Emotion Dataset" </a> from Kaggle.

<b> ✦ Data processing: </b> Text cleaning, tokenization.

<b> ✦ Building and training models: </b> Using CNN, RNN or LSTM for emotion recognition.

<b> ✦ Evaluation and improvement: </b> Evaluating model performance using metrics such as accuracy and improving the model.

### - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

### Loading the dataset, encoding the emotions and preparing the data for the model.

```javascript
def load_sentences():
    # Data collection and processing

    # Reading data
    df = pd.read_csv('Emotion_classify_Data.csv')

    # Emotions and their recoding
    label_encoder = LabelEncoder()
    df['Emotion_ENCOD'] = label_encoder.fit_transform(df['Emotion'])
    labels = np.array(to_categorical(df['Emotion_ENCOD']))

    emociniu_kategorijos_ir_statistika = df.groupby(['Emotion_ENCOD', 'Emotion'])[  #emotional_categories_and_statistics
        ['Emotion', 'Emotion_ENCOD']].size().reset_index().rename(columns={0: 'Count'})
    print('Categories of emotional sentences and their statistics:')
    print(emociniu_kategorijos_ir_statistika)

    unique_emotion_names = emociniu_kategorijos_ir_statistika['Emotion'].to_list()

    # Dictionary
    tokenizer = Tokenizer(num_words=1000, oov_token='<OOV>')
    tokenizer.fit_on_texts(df['Comment'])

    # Transcoding sentences
    sequences = tokenizer.texts_to_sequences(df['Comment'])
    padded_sequences = pad_sequences(sequences, maxlen=20, padding='post', truncating='post')

    return tokenizer, padded_sequences, labels, unique_emotion_names


tokenizer, padded_sequences, labels, unique_emotion_names = load_sentences()
# Further split into training and testing samples
padded_seq_train, padded_seq_test, labels_train, labels_test = train_test_split(
    padded_sequences, labels, test_size=0.2, random_state=42
    )
```
![image](https://github.com/Samantjna/Emotion-Prediction-Using-NLP/assets/163418549/f624e2f5-357a-4ff1-af2b-14fc4dcdf267)

### Sequential model creation with Embedding, RNN (GRU type) and Dense layers.

```javascript
def create_model(optimizer='adam'):
    model = Sequential([
        Embedding(input_dim=1000, output_dim=10, input_length=20),
        GRU(60),
        Dense(3, activation='softmax')
    ])
    model.compile(optimizer=optimizer, loss='categorical_crossentropy', metrics=['accuracy'])
    return model
```

### Building and training a single model.

```javascript
def train_single_model(padded_seq_train, labels_train, epochs=100, batch_size=20):
    model = create_model()
    early_stopping = EarlyStopping(
        monitor="val_loss",
        min_delta=0,
        patience=10,
        verbose=1,
        mode="min",
        baseline=None,
        restore_best_weights=True,
        start_from_epoch=5,
    )
    log = model.fit(padded_seq_train, labels_train,
              epochs=epochs,
              batch_size=batch_size,
              callbacks=[early_stopping],
              validation_split=0.2)
    return model, log


model, log = train_single_model(padded_seq_train, labels_train, epochs=100, batch_size=20)
```

## Model evaluation

```javascript
#model_predict = model.predict(padded_seq_test)
test_loss, test_accuracy = model.evaluate(padded_seq_test, labels_test)

#Evaluating the model with data not used in training
print(f'Modelio ivertinimas su apmokyme nenaudotais duomenimis:\n'
      f' - nuostoliai: {test_loss:.3f}, \n '  #loss
      f'- tikslumas: {test_accuracy:.3f} ')  #accuracy
```
![paveikslas](https://github.com/Samantjna/Emotion-Prediction-Using-NLP/assets/1218781/7082f9b0-2ee9-401f-a0d9-fac135720ab6)
![paveikslas](https://github.com/Samantjna/Emotion-Prediction-Using-NLP/assets/1218781/a87d7685-49ac-4606-bfe2-da6f9b37b622)


### The user can test the performance himself (how the model recognizes emotions from text)

```javascript
def test_manually(model, unique_emotion_names):
    print(f'\nIšbandykite patys!') # 'try it yourself!'
    while True:
        test_tekstas = input(f'Iveskite teksta anglu kalba arba exit:') # 'Enter text in English or exit:'
        if test_tekstas == 'exit':
            break
        test_sequence = tokenizer.texts_to_sequences([test_tekstas])
        test_padded = pad_sequences(test_sequence, maxlen=20, padding='post', truncating='post')

        predictions = model.predict(test_padded)
        decoded_predictions = decode_emotion_name(unique_emotion_names, predictions)
        print(f'Priskirtas emocijos pavadinimas: {decoded_predictions}')

def decode_emotion_name(unique_emotions, prediction):
    # Decoding numerical results into emotion names
    max_idx = np.argmax(prediction)
    emotion_name = unique_emotions[max_idx]
    return emotion_name


test_manually(model, unique_emotion_names)
```
![paveikslas](https://github.com/Samantjna/Emotion-Prediction-Using-NLP/assets/1218781/cbc40058-d008-43ac-98eb-dde8f39ea2f1)


### Trying to look up hyperparameters (we got an error but couldn't resolve it)

```javascript
def run_grid_search(padded_seq_train, labels_train):
    model = KerasClassifier(model=create_model, verbose=0)

    param_grid = {
        'batch_size': sp_randint(10, 50),
        'epochs': sp_randint(10, 50),
        'optimizer': ['RMSprop', 'Adam']
    }

    random_search = RandomizedSearchCV(
        estimator=model,
        param_distributions=param_grid,
        n_jobs=-1,
        cv=3,
        n_iter=10,
        random_state=42)

    # ValueError: Sequential model 'sequential_10' has no defined outputs yet.
    # https://github.com/mrdbourke/tensorflow-deep-learning/discussions/256
    # https://github.com/tensorflow/tensorflow/releases/tag/v2.7.0 Breaking Changes
    # The methods Model.fit(), Model.predict(), and Model.evaluate() will no longer uprank input data of shape (batch_size,) to become (batch_size, 1).
    random_result = random_search.fit(padded_seq_train, labels_train)

    print(f"Best: {random_result.best_score_} using {random_result}")
    print('Geriausias rezultatas %.3f naudojant %s' % (random_result.best_score_, random_result.best_params_))

    return random_result.best_params_
```
### - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

# Emocijų-atpažinimas-iš-teksto-NLP (LT) <img src="https://media.giphy.com/media/3o7WIx7urV838kHFzW/giphy.gif" width="50">

<br>

## Įvadas
  Šis projektas susijęs su teksto emocijų atpažinimu, kuris gali būti naudojamas socialinės žiniasklaidos įrašų, klientų atsiliepimų ar pokalbių analizėje.
   Tikslas yra sukurti modelį, kuris nustatytų ir klasifikuotų įvairias emocijas tekste, pvz., džiaugsmą, pyktį ir baimę.

### - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

### Technologijos:
Python ✦ TensorFlow/Keras ✦ NLTK ✦ scikit-learn ✦ Pandas

### - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

## Darbo etapai:
<b> ✦ Duomenų rinkimas: </b> Naudojame duomenų rinkinį, tokį kaip <a href=https://www.kaggle.com/datasets/abdallahwagih/emotion-dataset/data>"Emotion Dataset" </a> iš Kaggle.

<b> ✦ Duomenų apdorojimas: </b> Teksto valymas, tokenizavimas..

<b> ✦ Modelių kūrimas ir mokymas: </b> CNN, RNN ar LSTM naudojimas emocijų atpažinimui.

<b> ✦ Vertinimas ir tobulinimas: </b> Modelio efektyvumo įvertinimas naudojant matavimo rodiklius, pavyzdžiui, tikslumą, ir modelio tobulinimas.

### - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

### Įkeliame duomenų rinkinį, užkoduojame emocijas ir paruošiame duomenis modeliui.

```javascript
def load_sentences():
    # Duomenų surinkimas ir apdorojimas

    # Nuskaitymas
    df = pd.read_csv('Emotion_classify_Data.csv')

    # Emocijos ir jų perkodavimas
    label_encoder = LabelEncoder()
    df['Emotion_ENCOD'] = label_encoder.fit_transform(df['Emotion'])
    labels = np.array(to_categorical(df['Emotion_ENCOD']))

    emociniu_kategorijos_ir_statistika = df.groupby(['Emotion_ENCOD', 'Emotion'])[
        ['Emotion', 'Emotion_ENCOD']].size().reset_index().rename(columns={0: 'Count'})
    print('Categories of emotional sentences and their statistics:')
    print(emociniu_kategorijos_ir_statistika)

    unique_emotion_names = emociniu_kategorijos_ir_statistika['Emotion'].to_list()

    # Žodynas
    tokenizer = Tokenizer(num_words=1000, oov_token='<OOV>')
    tokenizer.fit_on_texts(df['Comment'])

    # Sakinių perkodavimas
    sequences = tokenizer.texts_to_sequences(df['Comment'])
    padded_sequences = pad_sequences(sequences, maxlen=20, padding='post', truncating='post')

    return tokenizer, padded_sequences, labels, unique_emotion_names


tokenizer, padded_sequences, labels, unique_emotion_names = load_sentences()
# Papildomai padalinti į apmokymo ir testavimo imtis
padded_seq_train, padded_seq_test, labels_train, labels_test = train_test_split(
    padded_sequences, labels, test_size=0.2, random_state=42
    )
```
![image](https://github.com/Samantjna/Emotion-Prediction-Using-NLP/assets/163418549/f624e2f5-357a-4ff1-af2b-14fc4dcdf267)

### Sequential modelio su Embedding, RNN (GRU tipo) ir Dense sluoksniais sukūrimo funkcija.

```javascript
def create_model(optimizer='adam'):
    model = Sequential([
        Embedding(input_dim=1000, output_dim=10, input_length=20),
        GRU(60),
        Dense(3, activation='softmax')
    ])
    model.compile(optimizer=optimizer, loss='categorical_crossentropy', metrics=['accuracy'])
    return model
```

### Vieno modelio sukūrimas ir apmokymas.

```javascript
def train_single_model(padded_seq_train, labels_train, epochs=100, batch_size=20):
    model = create_model()
    early_stopping = EarlyStopping(
        monitor="val_loss",
        min_delta=0,
        patience=10,
        verbose=1,
        mode="min",
        baseline=None,
        restore_best_weights=True,
        start_from_epoch=5,
    )
    log = model.fit(padded_seq_train, labels_train,
              epochs=epochs,
              batch_size=batch_size,
              callbacks=[early_stopping],
              validation_split=0.2)
    return model, log


model, log = train_single_model(padded_seq_train, labels_train, epochs=100, batch_size=20)
```

## Modelio įvertinimas

```javascript
#model_predict = model.predict(padded_seq_test)
test_loss, test_accuracy = model.evaluate(padded_seq_test, labels_test)
print(f'Modelio ivertinimas su apmokyme nenaudotais duomenimis:\n'
      f' - nuostoliai: {test_loss:.3f}, \n '
      f'- tikslumas: {test_accuracy:.3f} ')
```
![paveikslas](https://github.com/Samantjna/Emotion-Prediction-Using-NLP/assets/1218781/7082f9b0-2ee9-401f-a0d9-fac135720ab6)
![paveikslas](https://github.com/Samantjna/Emotion-Prediction-Using-NLP/assets/1218781/a87d7685-49ac-4606-bfe2-da6f9b37b622)

# Vizualizacija

```javascript
def grafikai(log):
    # Tikslumas
    plt.figure()
    df = pd.DataFrame(log.history)
    df['accuracy'].plot(label='accuracy')
    df['val_accuracy'].plot(label='val_accuracy')
    plt.title('Modelio tikslumo kaita')
    plt.xlabel('Ciklas (epoch)')
    plt.ylabel('Tikslumas (accuracy)')
    plt.legend()
    plt.show()

    # Nuostoliai
    plt.figure()
    df['loss'].plot(label='loss')
    df['val_loss'].plot(label='val_loss')
    plt.title('Modelio nuostolių kaita')
    plt.xlabel('Ciklas (epoch)')
    plt.ylabel('Nuostoliai (loss)')
    plt.legend()
    plt.show()
```
<img src="https://github.com/Samantjna/Emotion-Prediction-Using-NLP/assets/163418549/2c57f2de-5d25-4388-8255-72e48672d984" 
     width="480" 
     height="360" />
<img src="https://github.com/Samantjna/Emotion-Prediction-Using-NLP/assets/163418549/3979e341-c575-4f26-81c0-a9689bd08cb1" 
     width="480" 
     height="360" />


### Naudotojas gali pats išsibandyti veikimą (kaip modelis atpažįsta emocijas iš teksto)

```javascript
def test_manually(model, unique_emotion_names):
    print(f'\nIšbandykite patys!')
    while True:
        test_tekstas = input(f'Iveskite teksta anglu kalba arba exit:')
        if test_tekstas == 'exit':
            break
        test_sequence = tokenizer.texts_to_sequences([test_tekstas])
        test_padded = pad_sequences(test_sequence, maxlen=20, padding='post', truncating='post')

        predictions = model.predict(test_padded)
        decoded_predictions = decode_emotion_name(unique_emotion_names, predictions)
        print(f'Priskirtas emocijos pavadinimas: {decoded_predictions}')

def decode_emotion_name(unique_emotions, prediction):
    # Skaitinių rezultatų dekodavimas į emocijų pavadinimus
    max_idx = np.argmax(prediction)
    emotion_name = unique_emotions[max_idx]
    return emotion_name


test_manually(model, unique_emotion_names)
```
![paveikslas](https://github.com/Samantjna/Emotion-Prediction-Using-NLP/assets/1218781/7779c1ad-f40e-45eb-849c-fbd9cc948e52)

### Bandymas ieškoti hiperparametrų (gavome klaidą, bet nepavyko išspręsti)

```javascript
def run_grid_search(padded_seq_train, labels_train):
    model = KerasClassifier(model=create_model, verbose=0)

    param_grid = {
        'batch_size': sp_randint(10, 50),
        'epochs': sp_randint(10, 50),
        'optimizer': ['RMSprop', 'Adam']
    }

    random_search = RandomizedSearchCV(
        estimator=model,
        param_distributions=param_grid,
        n_jobs=-1,
        cv=3,
        n_iter=10,
        random_state=42)

    # ValueError: Sequential model 'sequential_10' has no defined outputs yet.
    # https://github.com/mrdbourke/tensorflow-deep-learning/discussions/256
    # https://github.com/tensorflow/tensorflow/releases/tag/v2.7.0 Breaking Changes
    # The methods Model.fit(), Model.predict(), and Model.evaluate() will no longer uprank input data of shape (batch_size,) to become (batch_size, 1).
    random_result = random_search.fit(padded_seq_train, labels_train)

    print(f"Best: {random_result.best_score_} using {random_result}")
    print('Geriausias rezultatas %.3f naudojant %s' % (random_result.best_score_, random_result.best_params_))

    return random_result.best_params_
```
