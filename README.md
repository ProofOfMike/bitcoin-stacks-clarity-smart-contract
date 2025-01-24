# bitcoin-stacks-clarity-smart-contract
#Clarity #Smart #Contract for #Time-#Delayed #Token Distribution!!! This #Clarity #smart #contract automates the transfer of tokens to a specified recipient after a set time period. ⏱️ Ideal for use cases requiring scheduled token distribution, ensuring seamless automation for future dates.

## Smart Contracts

- Git Clone
``` bash
git clone https://github.com/ProofOfMike/bitcoin-stacks-clarity-smart-contract.git
```

- time-locked-wallet.clar
```
;; traits
;;

;; token definitions
;; 

;; constants
;; owner
(define-constant contract-owner tx-sender)

;; errors
(define-constant err-owner-only (err u100))
(define-constant err-already-locked (err u101))
(define-constant err-unlock-in-past (err u102))
(define-constant err-no-value (err u103))
(define-constant err-beneficiary-only (err u104))
(define-constant err-unlock-height-not-reached (err u105))

;; data vars
(define-data-var beneficiary (optional principal) none)
(define-data-var unlock-height uint u0)

;; data maps
;;

;; public functions
;; The lock function does nothing more than transferring some 
;; tokens from the tx-sender to itself and setting the two variables.
;; However, we must not forget to check if the proper conditions
;; are set. Specifically:
(define-public (lock (new-beneficiary principal) (unlock-at uint) (amount uint)) 
  (begin 
    (asserts! (restricted tx-sender) err-owner-only)
    (asserts! (is-none (var-get beneficiary)) err-already-locked)
    (asserts! (> unlock-at block-height) err-unlock-in-past)
    ;; The (as-contract tx-sender) part gives us the principal of the contract.
    (try! (stx-transfer?  amount tx-sender (as-contract tx-sender)))
    (var-set beneficiary (some new-beneficiary))
    (var-set unlock-height unlock-at)
    (ok true)
  )
)

;; The bestow function will be straightforward. 
;; It checks if the tx-sender is the current beneficiary, 
;; and if so, will update the beneficiary to the passed principal. 
;; One side-note to keep in mind is that the principal is stored 
;; as an (optional principal). We thus need to wrap the tx-sender
;; in a (some ...) before we do the comparison.
(define-public (bestow (new-beneficiary principal)) 
  (begin  
     (asserts! (check tx-sender) err-beneficiary-only)
     (var-set beneficiary (some new-beneficiary))
     (ok true)
  )
)

;; claim function should check if both the tx-sender is the 
;; beneficiary and that the unlock height has been reached.
(define-public (claim) 
  (begin 
     (asserts! (check tx-sender) err-beneficiary-only)
     (asserts! (>= block-height (var-get unlock-height)) err-unlock-height-not-reached)
     ;; we didin't ok response here because stx-transfer returns a response.
     (as-contract 
       (stx-transfer? 
        (stx-get-balance tx-sender)
        tx-sender
        (unwrap-panic (var-get beneficiary))
        )
     )
  )
)


;; read only functions
;;

;; private functions
(define-private (restricted (caller principal)) 
  (is-eq caller contract-owner)
)

(define-private (check (caller principal)) 
  (is-eq (some caller) (var-get beneficiary))
)
```

## Contact Info

X: [@mikegraham915](https://x.com/@mikegraham915)


