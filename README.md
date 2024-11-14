# Next-Word-Prediction-using-LSTM

## AIM

To develop an LSTM-based model for predicting the next word in a text corpus.

## Problem Statement and Dataset

Build a neural network-based text generator to produce new sequences from a given corpus by tokenizing text, creating n-grams, padding sequences, and encoding labels.
![image](https://github.com/user-attachments/assets/18b913bd-c192-4f49-83ee-741f69a36158)

## DESIGN STEPS

### STEP 1:
Use fit_vectorizer to initialize and fit a TextVectorization layer on the corpus for word-to-integer tokenization.
### STEP 2:
Generate n-grams for each sentence using n_gram_seqs, creating sequential input data.
### STEP 3:
Pad these sequences to a uniform length with pad_seqs, enabling consistent input shapes for training.
### STEP 4:
Split each sequence into features and labels, where features contain all words except the last, and labels are the last word.
### STEP 5:
One-hot encode the labels with a vocabulary size from total_words for categorical prediction.
### STEP 6:
Construct a TensorFlow dataset with these features and labels, batching them for efficient processing.
### STEP 7:
Build the model with an Embedding layer, Bidirectional LSTM for sequence processing, and Dense layer with softmax for word prediction.
### STEP 8:
Compile and train the model using categorical cross-entropy loss and the Adam optimizer.

## PROGRAM
### Name: GURUMOORTHI R
### Register Number: 212222230042

### 1.fit_vectorizer function

```py

def fit_vectorizer(corpus):
    """
    Instantiates the vectorizer class on the corpus

    Args:
        corpus (list): List with the sentences.

    Returns:
        (tf.keras.layers.TextVectorization): an instance of the TextVectorization class containing the word-index dictionary, adapted to the corpus sentences.
    """

    tf.keras.utils.set_random_seed(65) # Do not change this line or you may have different expected outputs throughout the assignment

   
    vectorizer = tf.keras.layers.TextVectorization(
         standardize='lower_and_strip_punctuation',                 # Convert to lowercase and strip punctuation
        split= 'whitespace' ,                      # Split on whitespace (default)
        ragged= True   ,                    # Allow ragged tensors
        output_mode= 'int'                 # Output as integers
    )

    # Adapt it to the corpus
    vectorizer.adapt(corpus)


    return vectorizer
```

### 2. n_grams_seqs function
```py

def n_gram_seqs(corpus, vectorizer):

    """
    Generates a list of n-gram sequences

    Args:
        corpus (list of string): lines of texts to generate n-grams for
        vectorizer (tf.keras.layers.TextVectorization): an instance of the TextVectorization class adapted in the corpus

    Returns:
        (list of tf.int64 tensors): the n-gram sequences for each line in the corpus
    """
    input_sequences = []


    for sentence in corpus:
        # Vectorize the sentence to get the token indices
        vectorized_sentence =vectorizer(sentence)

        # Generate n-grams for the vectorized sentence
        for i in range(2,len(vectorized_sentence)+1):  # Start from 2 to avoid the first token
            n_gram = vectorized_sentence[:i+1]
            # Append the n-gram to the list
            input_sequences.append( n_gram)

    return input_sequences
```

### 3. pad_seqs function
```py

def pad_seqs(input_sequences, max_sequence_len):

    """
    Pads tokenized sequences to the same length

    Args:
        input_sequences (list of int): tokenized sequences to pad
        maxlen (int): maximum length of the token sequences

    Returns:
        (np.array of int32): tokenized sequences padded to the same length
    """

   
    input_list = [seq if isinstance(seq, list) else seq.numpy().tolist() for seq in input_sequences]

    # Use pad_sequences to pad the sequences with left padding ('pre')
    padded_sequences = tf.keras.preprocessing.sequence.pad_sequences(
        input_list,
        maxlen=max_sequence_len,
        padding='pre',
        dtype='int32'
                     # Use the list of lists for padding
                     # Set the maximum length
                     # Pad sequences to the left (before the sequence)
                     # Specify the output type as int32
    )


    return padded_sequences

```

### 4. features_and_labels_dataset function
```py

def features_and_labels_dataset(input_sequences, total_words):
    """
    Generates features and labels from n-grams and returns a tensorflow dataset

    Args:
        input_sequences (list of int): sequences to split features and labels from
        total_words (int): vocabulary size

    Returns:
        (tf.data.Dataset): Dataset with elements in the form (sentence, label)
    """
 
    # Define the features by taking all tokens except the last one for each sequence
    features = input_sequences[:,:-1]
    labels = input_sequences[:,-1]
    # One-hot encode the labels using total_words as the number of classes
    one_hot_labels = tf.keras.utils.to_categorical(labels,num_classes=total_words)
    # Build the dataset using the features and one-hot encoded labels
    dataset = tf.data.Dataset.from_tensor_slices((features,one_hot_labels))

    # Batch the dataset with a batch size of 16
    batch_size = 16  # Feel free to adjust this based on the global variable, but should be <= 64
    batched_dataset = dataset.batch(batch_size)



    return batched_dataset

```

### 5.create_model function
```py
 
def create_model(total_words, max_sequence_len):
    """
    Creates a text generator model

    Args:
        total_words (int): size of the vocabulary for the Embedding layer input
        max_sequence_len (int): length of the input sequences

    Returns:
       (tf.keras Model): the text generator model
    """
    model = tf.keras.Sequential()

    model.add(tf.keras.layers.Input(shape=(max_sequence_len-1,)))
    model.add(tf.keras.layers.Embedding(total_words,EMBEDDING_DIM))
    model.add(tf.keras.layers.Bidirectional(tf.keras.layers.LSTM(150)))
    model.add(tf.keras.layers.Dense(total_words, activation='softmax'))

    # Compile the model
    model.compile(
    loss='categorical_crossentropy',
    optimizer="adam",
    metrics=['accuracy']
    )
    
 

    return model
```



## OUTPUT
### 1. fit_vectorizer output

![1 1](https://github.com/user-attachments/assets/64b950e8-eacf-4acd-aa84-cb288e45151b)

![1 2](https://github.com/user-attachments/assets/129baccd-8be0-428d-8595-0a573bb4b228)


### 2. n_grams_seqs output

![2](https://github.com/user-attachments/assets/4422673b-7ac0-4664-8bc3-e35487ad5c03)


### 3. pad_seqs output

![3](https://github.com/user-attachments/assets/30b9fff0-ad13-404e-95b7-d1b9040c4d9a)


### 4. features_and_labels_dataset output

![4](https://github.com/user-attachments/assets/7b53877c-3d8c-4ee2-ae80-0e83376e5214)


### 5. Training Loss, Validation Loss Vs Iteration Plot

![5](https://github.com/user-attachments/assets/b501a51e-de74-49df-bac7-0730ef3c87a1)



### 6. Sample Text Prediction

![6](https://github.com/user-attachments/assets/8adb309a-b4a5-4025-a88d-7ef55392f258)


## RESULT

Thus, a trained text generator model capable of predicting the next word in a sequence from the given corpus is successfully implemented.
