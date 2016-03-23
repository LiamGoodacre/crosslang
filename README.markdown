*Project not yet started.*

The goal is to define a common interface for generating data structures and functions for building and processing them across many different languages.


*Possible features:*

- Records
- Unions
- Type parameters
  - Higher kinded for descriptions
  - Specialization to non-higher kinded for output (e.g. no support in Java)
- Pattern matching
- Direct immutable manipulation
  - Getters
  - Setters
  - Modifiers
- Optics
  - Fold
  - Traversal
  - Lens
  - Prism


*Language Roadmap:*

- Java
- JavaScript
- Python
- PureScript
- Haskell


*Initial idea/example:*

~~~haskell
context ProductTypes

  -- | a product type with fst and snd fields
  Record Prod (a : *) (b : *)
    fst : a
    snd : b

  -- pattern matching
  match ← Match Prod

  -- set a field to a new value
  setFst ← Setter Prod.fst
  setSnd ← Setter Prod.snd

  -- modify a field by a mono-morphic function
  -- not needed here (see following functor functions)
  -- might be useful on non-polymorphic types
  modifyFst ← Modify Prod.fst
  modifySnd ← Modify Prod.snd

  -- mapping functions based on type arguments
  mapFst ← Functor (Prod _ !)
  mapSnd ← Functor (Prod ! _)
  mapBoth ← Functor (Prod _ _)

  -- optics
  fst ← Lens Prod.fst
  snd ← Lens Prod.snd
  both ← RecordTraversal [Prod.fst, Prod.snd]


context SumTypes

  -- | a sum type with left or right values
  Union Sum (a : *) (b : *)
    Left  : forall a. a → Sum a b
    Right : forall b. b → Sum a b

  -- pattern matching
  match ← Match Sum

  -- mapping functions based on type arguments
  mapLeft ← Map (Sum _ !)
  mapRight ← Map (Sum ! _)
  mapBoth ← Map (Sum _ _)

  -- optics
  left ← Prism (Sum.Left _)
  right ← Prism (Sum.Right _)
  either ← UnionTraversal [Sum.Left _, Sum.Right _]
~~~


*Java interpretation*

~~~java
/**
 * @summary a product type with fst and snd fields
 */
public final class ProductTypes {

  public static final class Prod<a, b> {
    public final a fst;
    public final b snd;

    private Prod(a fst, b snd) {
      this.fst = fst;
      this.snd = snd;
    }

    public static <a, b> Prod<a, b> create(final a fst,
                                           final b snd) {
      return new Prod(fst, snd);
    }
  }

  public static <a, b> Prod<a, b> create(final a fst,
                                         final b snd) {
    return Prod.<a, b>create(fst, snd);
  }

  public static <a, b, c> c match(final Function2<a, b, c> matcher,
                                  final Prod<a, b> value) {
    return matcher.apply(value.fst, value.snd);
  }

  public static <a, b, c> Prod<c, b> setFst(final c fst,
                                            final Prod<a, b> value) {
    return create(fst, value.snd);
  }

  public static <a, b, c> Prod<a, c> setSnd(final c snd,
                                            final Prod<a, b> value) {
    return create(value.fst, snd);
  }

  public static <a, b> Prod<a, b> modifyFst(final Function1<a, a> fn,
                                            final Prod<a, b> value) {
    return create(fn.apply(value.fst), value.snd);
  }

  public static <a, b> Prod<a, b> modifySnd(final Function1<b, b> fn,
                                            final Prod<a, b> value) {
    return create(value.fst, fn.apply(value.snd));
  }

  public static <a, b, c> Prod<c, b> mapFst(final Function1<a, c> fn,
                                            final Prod<a, b> value) {
    return create(fn.apply(value.fst), value.snd);
  }

  public static <a, b, c> Prod<a, c> mapSnd(final Function1<b, c> fn,
                                            final Prod<a, b> value) {
    return create(value.fst, fn.apply(value.snd));
  }

  public static <a, b> Prod<b, b> mapBoth(final Function1<a, b> fn,
                                          final Prod<a, a> value) {
    return create(fn.apply(value.fst), fn.apply(value.snd));
  }

  public static <a, b, c> Lens<Prod<a, b>, Prod<c, b>, a, c> fst() {
    return new Lens<Prod<a, b>, Prod<c, b>, a, c>() {
      public a get(final Prod<a, b> value) {
        return value.fst;
      }
      public Prod<c, b> modify(final Function1<a, c> fn,
                            final Prod<a, b> value) {
        return create(fn.apply(value.fst), value.snd);
      }
    };
  }

  public static <a, b, c> Lens<Prod<a, b>, Prod<a, c>, b, c> snd() {
    return new Lens<Prod<a, b>, Prod<a, c>, b, c>() {
      public b get(final Prod<a, b> value) {
        return value.snd;
      }
      public Prod<a, c> modify(final Function1<b, c> fn,
                            final Prod<a, b> value) {
        return create(value.fst, fn.apply(value.snd));
      }
    };
  }

  public static <a, b, c> Traversal<Prod<a, a>, Prod<b, b>, a, b> both() {
    return new Traversal<Prod<a, a>, Prod<b, b>, a, b>() {
      // TODO
    };
  }
}

// TODO SumTypes
~~~


*JavaScript interpretation*

~~~javascript
var ProductTypes = (function () {
  var ProductTypes = {};

  var Prod = function Prod(fst, snd) {
    this.fst = fst;
    this.snd = snd;
  };
  Prod.create = function (fst, snd) {
    return new Prod(fst, snd);
  };

  ProductTypes.create = Prod.create;

  ProductTypes.match = function (matcher, value) {
    return matcher(value.fst, value.snd);
  };

  ProductTypes.setFst = function (fst, value) {
    return ProductTypes.create(fst, value.snd);
  };

  ProductTypes.setSnd = function (snd, value) {
    return ProductTypes.create(value.fst, snd);
  };

  ProductTypes.modifyFst = function (fn, value) {
    return ProductTypes.create(fn(value.fst), value.snd);
  };

  ProductTypes.modifySnd = function (fn, value) {
    return ProductTypes.create(value.fst, fn(value.snd));
  };

  ProductTypes.mapFst = function (fn, value) {
    return ProductTypes.create(fn(value.fst), value.snd);
  };

  ProductTypes.mapSnd = function (fn, value) {
    return ProductTypes.create(value.fst, fn(value.snd));
  };

  ProductTypes.mapBoth = function (fn, value) {
    return ProductTypes.create(fn(value.fst), fn(value.snd));
  };

  ProductTypes.fst = Lens.create(ProductTypes.getFst,
                                 ProductTypes.mapFst);

  ProductTypes.snd = Lens.create(ProductTypes.getSnd,
                                 ProductTypes.mapSnd);

  ProductTypes.both = Traversal.create(/*TODO*/);

  return ProductTypes;
})();

// TODO SumTypes
~~~