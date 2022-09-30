# FirebaseGoogleSignIn

#@ Step -1 -> Connect android studio  to Firebase 
  
  #@ Step-2 ->  Add required dependency to android project . 
    
    implementation 'com.google.android.gms:play-services-auth:18.0.0'
    implementation 'com.google.firebase:firebase-auth:19.3.2'
    
  #@ Step-3 -> Add google sign in button to the xml 
  
  <com.google.android.gms.common.SignInButton
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:id="@+id/btn_google_sign_In"
        app:layout_constraintTop_toTopOf="parent"
        android:layout_marginLeft="17dp"
        app:layout_constraintStart_toStartOf="parent"
        android:layout_marginRight="17dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        />

#@ Step-4 -> Create FirebaseAuth global object

        private FirebaseAuth auth;// outside the onCreate function
        auth= FirebaseAuth.getInstance(); //  inside onCreate Function
        
        // continues in that function following : 
        
        GoogleSignInOptions gso= new GoogleSignInOptions.Builder(GoogleSignInOptions.DEFAULT_SIGN_IN)
                .requestIdToken(getString(R.string.default_web_client_id))
                .requestEmail()
                .build();

        //Build a GoogleSignInClient with the options secified by gso .
        googleSignInClient= GoogleSignIn.getClient(this,gso);

        

 #@ Step-5 -> Add following in corresponding java file  in  onClick google sign in button 
 
 Intent signInIntent= googleSignInClient.getSignInIntent();
        startActivityForResult(signInIntent, 101);
        
 #@ Step-6 ->  override onActivityResult(....) outside the  onCreate() function 
     @Override
    protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        // Result returned from Launching the Intent from GoogleSignInApi.getSignInIntent(...);
        if(requestCode == 101){
            Task<GoogleSignInAccount> task= GoogleSignIn.getSignedInAccountFromIntent(data);
            try{
                // Google Sign In was successful ,  authenticate with Firebase
                GoogleSignInAccount account= task.getResult(ApiException.class);
                firebaseAuthWithGoogle(account.getIdToken());
            }
            catch (ApiException e){
                Snackbar.make( mainBinding.mainLayout, "Exception is occuring "+ e.getMessage(), BaseTransientBottomBar.LENGTH_LONG)
                        .show();
            }
        }

#@ Step -7 ->  write code  of  firebaseAuthWithGoogle()


    private void firebaseAuthWithGoogle(String idToken) {
        AuthCredential credential= GoogleAuthProvider.getCredential(idToken, null);
        auth.signInWithCredential(credential)
                .addOnCompleteListener(this, new OnCompleteListener<AuthResult>() {
                    @Override
                    public void onComplete(@NonNull Task<AuthResult> task) {
                        if(task.isSuccessful()){
                            // Sign in success , update UI .
                            Log.d(TAG, "onComplete:  success");
                            FirebaseUser user= auth.getCurrentUser();
                            updateUI(user);
                        }
                        else {
                            Log.w(TAG, "onComplete: ", task.getException());
                            Snackbar.make(mainBinding.mainLayout, "Authentication Failed.", BaseTransientBottomBar.LENGTH_LONG)
                                    .setAction("UNDO",
                                            // If undo button is pressed the toast message
                                            new View.OnClickListener() {
                                                @Override
                                                public void onClick(View view) {
                                                    Toast.makeText(MainActivity.this, "Undo is Clicked . ", Toast.LENGTH_SHORT).show();
                                                }
                                            })
                                    .show();
                            updateUI(null);
                        }
                    }
                });
    }


 #@ Step -7 -> Optional ->  If user previously logged in then we should move to dashboard then outside onCreate() function  override onStart() function 
 
 
    @Override
    protected void onStart() {
        super.onStart();
        //  Check if user is signed in (non-null)  and update UI accordinly .
        FirebaseUser currentUser= auth.getCurrentUser();
        updateUI(currentUser);
        
        
#@ Step -B -> In Dashboard activity 
   1->  Create GoogleSignInAccount object 
      GoogleSignInAccount account= GoogleSignIn.getLastSignedInAccount(this);
      
          a-> to get user image -> account.getPhotoUrl() 
          b-> to get user name  -> account.getDisplayName()
          c-> to get user email -> account.getEmail()

  2-> To sign out  on sign out button click on dashboard 
      
        FirebaseAuth.getInstance().signOut();
        
  
