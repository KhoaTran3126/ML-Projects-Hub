# General Code Templates

## Cross-Validation
<pre>
  <code>
def cross_validate(model, model_type, X, y, scorer, n_splits=5):
    kfold  = KFold(n_splits=n_splits, shuffle=True, random_state=3126)
    scores = np.zeros(n_splits) 

    for i,(train_idx,val_idx) in enumerate(kfold.split(X)):
        X_train, y_train = X.iloc[train_idx,:], y[train_idx]
        X_val,   y_val   = X.iloc[val_idx,:],   y[val_idx]

        cloned_model = clone(model)
        ## Fits model
        if model_type=="lgbm":
            cloned_model.fit(
                X_train, y_train,
                eval_set=[(X_val, y_val)],
                callbacks=[lightgbm.early_stopping(100, verbose=False)])
      
        elif model_type=="xgboost":
            cloned_model.fit(
                X_train, y_train,
                eval_set=[(X_val, y_val)],
                early_stopping_rounds=100, verbose=False)
            
        elif model_type=="catboost":
            pass
    
        else: 
            cloned_model.fit(X_train, y_train)

        ## Stores the score
        scores[i] = scorer(y_val, cloned_model.predict(X_val))
    return scores
  </code>
</pre>

## Optuna
<pre>
  <code>
def objective(trial):
    params = {...}
    
    model  = Model(**params)
    scores = cross_validate(Model, "model_name", X, y, scorer)
    return np.mean(scores)

study = optuna.create_study(
    direction="minimize",
    sampler=optuna.samplers.TPESampler(seed=3126),
    pruner=optuna.pruners.MedianPruner(n_warmup_steps=15)
)
study.optimize(objective, n_trials=100)
  </code>
</pre>

## SHAP values Analysis for Tree Models
<pre>
  <code>
explainer   = shap.TreeExplainer(model)
shap_values = explainer.shap_values(X_val)

shap.summary_plot(shap_values, X_val, plot_type="bar", plot_size=(16,4))
shap.summary_plot(shap_values, X_val)

top_predictive_features = [...]
for feature in top_predictive_features:
    shap.dependence_plot(
        feature,
        shap_values,
        X_val,
        interaction_index=None
    )
  </code>
</pre>
