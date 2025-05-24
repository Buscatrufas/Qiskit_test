import numpy as np
from math import gcd
from fractions import Fraction
from qiskit import Aer, QuantumCircuit, transpile

def shor_factor_15():
    N = 15
    a = 7  # coprimo con 15
    n_count = 4  # 2^4 = 16 > 15^2, suficiente para fase
    simulator = Aer.get_backend('aer_simulator')

    def controlled_mult_mod15(qc, control_qubit, target_qubits, a_exp):
        # hardcoded for a^x mod 15 with a = 7
        if a_exp % 15 == 7:
            qc.cx(control_qubit, target_qubits[0])
            qc.cx(control_qubit, target_qubits[1])
        elif a_exp % 15 == 4:
            qc.cx(control_qubit, target_qubits[1])
            qc.cx(control_qubit, target_qubits[2])
        elif a_exp % 15 == 13:
            qc.cx(control_qubit, target_qubits[0])
            qc.cx(control_qubit, target_qubits[2])
        elif a_exp % 15 == 1:
            pass  # identity

    def qft_dagger(qc, n):
        for qubit in range(n//2):
            qc.swap(qubit, n - qubit - 1)
        for j in range(n):
            for m in range(j):
                qc.cp(-np.pi/float(2**(j - m)), m, j)
            qc.h(j)

    def get_order_from_phase(phase, max_denominator=15):
        frac = Fraction(phase).limit_denominator(max_denominator)
        return frac.denominator

    def try_get_factors_from_r(a, N, r):
        if r % 2 != 0:
            return None
        x = pow(a, r // 2, N)
        if x == N - 1 or x == 1:
            return None
        p = gcd(x - 1, N)
        q = gcd(x + 1, N)
        if p * q == N and p != 1 and q != 1:
            return sorted([p, q])
        return None

    max_attempts = 10
    for attempt in range(1, max_attempts + 1):
        # Crear circuito
        qc = QuantumCircuit(n_count + 4, n_count)
        qc.h(range(n_count))  # superposición en control
        qc.x(n_count)         # target inicializado en 1

        for q in range(n_count):
            a_exp = pow(a, 2**q, N)
            controlled_mult_mod15(qc, q, [n_count, n_count+1, n_count+2, n_count+3], a_exp)

        qft_dagger(qc, n_count)
        qc.measure(range(n_count), range(n_count))

        qc_compiled = transpile(qc, simulator)
        result = simulator.run(qc_compiled).result()
        counts = result.get_counts()
        measured_bin = max(counts, key=counts.get)
        phase = int(measured_bin, 2) / (2**n_count)

        print(f"\nIntento #{attempt}")
        print(f"Medición: {measured_bin} (fase ~ {phase:.4f})")

        r = get_order_from_phase(phase)
        factors = try_get_factors_from_r(a, N, r)
        
        if factors:
            print(f"🎉 ¡Factores encontrados!: {factors}")
            return factors

    print("\n❌ No se encontraron factores útiles tras múltiples intentos.")
    return None

# Ejecutar la PoC
if __name__ == "__main__":
    shor_factor_15()
