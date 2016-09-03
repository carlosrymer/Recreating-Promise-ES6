```javascript
// Recreating Promise in ES6
class Promise {
  constructor(callback) {
    this._errorCallbacks   = [];
    this._successCallbacks = [];

    if (typeof callback === 'function') {
      try {
        callback(this.resolveIt.bind(this), this.rejectIt.bind(this));
      } catch (e) {
        this.rejectIt(e);
      }
    }

    return this;
  }

  catch(callback) {
    this._errorCallbacks.push((...args) => {
      let result;

      try {
        result = callback.apply(this, args);
      } catch (e) {
        return this.rejectIt(e);
      }

      return this.resolveIt(result);
    });

    return this;
  }

  rejectIt(error) {
    let result;
    let callback;

    if (typeof this._errorCallbacks[0] === 'function') {
      callback = this._errorCallbacks.shift();

      try {
        result = callback.call(this, error);
      } catch (e) {
        this.rejectIt(e);
      }
    } else {
      throw error;
    }
  }

  resolveIt(value) {
    let result;
    let callback;

    if (typeof this._successCallbacks[0] === 'function') {
      callback = this._successCallbacks.shift();

      try {
        result = callback.call(this, value);
      } catch (e) {
        this.rejectIt(e);
      }
    }

    return value;
  }

  static reject(error) {
    return new Promise((resolve, reject) => {
      reject(error);
    });
  }

  static resolve(result) {
    return new Promise(resolve => {
      resolve(result);
    });
  }

  then(callback) {
    this._successCallbacks.push((...args) => {
      let result;

      try {
        result = callback.apply(this, args);
      } catch (e) {
        return this.rejectIt(e);
      }

      return this.resolveIt(result);
    });

    return this;
  }
}

// Usage
new Promise((resolve, reject) => setTimeout(() => resolve('OK'), 1000))
  .then(result => {
    console.log(result);

    return result;
  })
  .then(result => {
    console.log(result);

    return result;
  });
  
Promise.resolve('OK')
  .then(result => {
    console.log(result);

    return result;
  })
  .then(result => {
    console.log(result);

    return result;
  });
  
Promise.reject(new Error('OK'))
  .catch(e => {
    console.log(e.message);

    return e.message;
  })
  .then(result => {
    console.log(result);

    return result;
  });
```
