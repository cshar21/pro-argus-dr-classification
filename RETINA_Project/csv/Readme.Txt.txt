..in this case data augmentation was not applied during pre processing 
only clahe 

resizing was not applied here to give you the option to switch to other models if you want to

data augmentation needs to be applied during training because :

Static Augmentations: The model will see the exact same augmented version of an image every epoch → defeats the purpose of augmentation (no randomness).

Apply augmentations dynamically during training via Albumentations in the Dataset class → new randomness every epoch.



ISSUES :
1. Augmentations Should Happen During Training, Not Preprocessing 
   Changes : Remove all augmentations from get_augmentations() in preprocessing.
Only apply augmentations dynamically during training 

2. Aggressive Spatial Transforms Might Distort Retinal Lesions 
   need to reconfirm the correct values for augmentation later

3. Resize Should Happen During Training, Not Preprocessing 
   to be able to switch to other models later without pre processing again

4. Normalization Issue in Preprocessing
🔴 Issue:

Right now, ToTensorV2() in preprocessing converts images to [0,1] range, but later you multiply by 255 to save them as PNGs.
This might cause floating-point conversion artifacts
✅ Fix:

Do not use ToTensorV2() in preprocessing.
Save images as uint8 without normalization.
Normalize dynamically in your DataLoader during training.

5. Check train/val split to avoid data leakage. 
   even though i have APTOS test images and csv why do a train / val split ?
   🔍 Why Validation is Still Important
1️⃣ Prevents Overfitting:

Without validation, you have no way to monitor when your model is overfitting on the training data.
Loss decreasing ≠ better model! Sometimes, the model memorizes instead of generalizing.
2️⃣ Helps Tune Hyperparameters:

Learning rate schedules, augmentation strategies, and early stopping all depend on a validation set.
3️⃣ Avoids Surprises on APTOS Test Set:

If your model fails on the APTOS test set, you won’t know why (bad training? overfitting?).
A validation set lets you debug issues before the final test.
✅ Option 1 (Recommended): Split Train/Val Within Each Dataset

Take 10-20% of each dataset (APTOS, EyePACS, Messidor) for validation.
Use stratified split (so all severity levels are balanced).
Train on remaining 80-90%.
✅ Option 2 (Alternative): Use One Dataset for Validation

Train on EyePACS + Messidor + 90% APTOS
Validate on 10% of APTOS
Test on APTOS Test Set
✅ Option 3 (Risky): No Validation, Just Train & Test on APTOS
⚠ This means no early stopping, no fine-tuning, no debugging.
⚠ If the model performs badly on APTOS, you won’t know what went wrong.





Create train-validation split dynamically when loading data during training.
Use Stratified Split (train_test_split(stratify=y))
OR Messidor as validation
OR K-Fold Cross-Validation




📝 What You Need to Do
1️⃣ Modify Preprocessing Script

Remove all augmentations except CLAHE
Do not resize (keep original image size)
Do not use ToTensorV2() in preprocessing
Save images as uint8 PNGs (not float)
2️⃣ Modify Training Dataset

Apply Albumentations during training inside __getitem__()
Dynamically resize inside the dataset
Normalize inside the dataset


