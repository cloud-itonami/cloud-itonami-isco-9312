(ns civil-labour.store
  "SSoT for the ISCO-08 9312 independent civil-labour-crew sole-proprietor
  actor, behind a `Store` protocol so the backend is a swap (MemStore
  default ‖ a real Datomic/kotoba-server backend, per the itonami actor
  pattern).

  Domain = independent civil engineering labour crew operations:

    site        — a work site (siteId, hasExcavation? boolean)
    work-order  — a labour order scoped to a site (orderId, siteId, scope)
    labor-task  — a task performed under a work order (taskId, orderId, kind
                  #{:standard :near-excavation}, performedBy #{:robot :crew})
    inspection  — a clearance inspection result (inspectionId, orderId,
                  result #{:pass :fail})

  The append-only records are the operating ledger: a labor task or
  inspection must reference a registered work order on a registered site,
  and labor-tasks/inspections are never mutated in place, only appended.")

(defprotocol Store
  (site [st site-id])
  (work-order [st order-id])
  (work-orders-of [st site-id])
  (labor-tasks-of [st order-id])
  (inspections-of [st order-id])
  (register-site! [st site])
  (register-work-order! [st work-order])
  (record-labor-task! [st labor-task])
  (record-inspection! [st inspection]))

(defrecord MemStore [state]
  Store
  (site [_ site-id]
    (get-in @state [:sites site-id]))
  (work-order [_ order-id]
    (get-in @state [:work-orders order-id]))
  (work-orders-of [_ site-id]
    (filter #(= site-id (:site-id %)) (vals (:work-orders @state))))
  (labor-tasks-of [_ order-id]
    (filter #(= order-id (:order-id %)) (:labor-tasks @state)))
  (inspections-of [_ order-id]
    (filter #(= order-id (:order-id %)) (:inspections @state)))
  (register-site! [_ site]
    (swap! state assoc-in [:sites (:site-id site)] site))
  (register-work-order! [_ work-order]
    (swap! state assoc-in [:work-orders (:order-id work-order)] work-order))
  (record-labor-task! [_ labor-task]
    (swap! state update :labor-tasks (fnil conj []) labor-task))
  (record-inspection! [_ inspection]
    (swap! state update :inspections (fnil conj []) inspection)))

(defn mem-store
  ([] (mem-store {}))
  ([seed]
   (->MemStore (atom (merge {:sites {} :work-orders {} :labor-tasks [] :inspections []} seed)))))
