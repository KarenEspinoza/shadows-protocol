# shadows-protocol
Desarrollo de videojuego II. Actividad 1
using UnityEngine; 
public class PlayerMovement : MonoBehaviour
{
    public CharacterController controller;
    public float speed = 6f;
    public float gravity = -9.81f;
    public float jumpHeight = 1.5f;
    public Transform groundCheck;
    public float groundDistance = 0.4f;
    public LayerMask groundMask;

    Vector3 velocity;
    bool isGrounded;

    void Update()
    {
        isGrounded = Physics.CheckSphere(groundCheck.position, groundDistance, groundMask);
        if (isGrounded && velocity.y < 0)
            velocity.y = -2f;

        float x = Input.GetAxis("Horizontal");
        float z = Input.GetAxis("Vertical");
        Vector3 move = transform.right * x + transform.forward * z;
        controller.Move(move * speed * Time.deltaTime);

        if (Input.GetButtonDown("Jump") && isGrounded)
            velocity.y = Mathf.Sqrt(jumpHeight * -2f * gravity);
 
        velocity.y += gravity * Time.deltaTime;
        controller.Move(velocity * Time.deltaTime);
    }
}

//Interacción con el entorno
using UnityEngine;
 
public class Interact : MonoBehaviour
{
    public float range = 3f;
    public Camera fpsCam;

    void Update()
    {
        if (Input.GetKeyDown(KeyCode.E))
        {
            Ray ray = new Ray(fpsCam.transform.position, fpsCam.transform.forward);
            RaycastHit hit;
            if (Physics.Raycast(ray, out hit, range))
            {
                IInteractable interactable = hit.collider.GetComponent<IInteractable>();
                if (interactable != null)
                    interactable.Interact();
            }
        }
    }
}

public interface IInteractable
{
    void Interact();
}

// SISTEMA DE COMBATE
using UnityEngine;

public class Gun : MonoBehaviour
{
    public float damage = 10f;
    public float range = 100f;
    public Camera fpsCam;
    public ParticleSystem muzzleFlash;
    public GameObject impactEffect;

    void Update()
    {
        if (Input.GetButtonDown("Fire1"))
            Shoot();
    }

    void Shoot()
    {
        muzzleFlash.Play();
        RaycastHit hit;
        if (Physics.Raycast(fpsCam.transform.position, fpsCam.transform.forward, out hit, range))
        {
           Target target = hit.transform.GetComponent<Target>();
            if (target != null)
                target.TakeDamage(damage);

            GameObject impactGO = Instantiate(impactEffect, hit.point, Quaternion.LookRotation(hit.normal));
            Destroy(impactGO, 2f);
        }
    }
}

public class Target : MonoBehaviour
{
    public float health = 50f;
    public void TakeDamage(float amount)
    {
        health -= amount;
        if (health <= 0f)
            Die();
    }
    void Die()
    {
        Destroy(gameObject);
    }
}

//INTELIGENCIA ARTIFICIAL ENEMIGA
using UnityEngine;
using UnityEngine.AI;

public class EnemyAI : MonoBehaviour
{
    public NavMeshAgent agent;
    public Transform[] patrolPoints;
    private int destPoint = 0;
    public Transform player;
    public float detectionRange = 10f;

    void Start()
    {
        GoToNextPoint();
    }

    void Update()
    {
        float distance = Vector3.Distance(player.position, transform.position);
 
        if (distance < detectionRange)
        {
            agent.SetDestination(player.position);
        }
        else if (!agent.pathPending && agent.remainingDistance < 0.5f)
        {
            GoToNextPoint();
        }
    }

    void GoToNextPoint()
    {
        if (patrolPoints.Length == 0)
            return;

        agent.destination = patrolPoints[destPoint].position;
        destPoint = (destPoint + 1) % patrolPoints.Length;
    }
}

//HUD del jugador
using UnityEngine;
using UnityEngine.UI;

public class PlayerHUD : MonoBehaviour
{
    public Slider healthBar;
    public Text ammoText;
    private PlayerStats stats;

    void Start()
    {
        stats = GetComponent<PlayerStats>();
    }

    void Update()
    {
        healthBar.value = stats.health;
        ammoText.text = "Ammo: " + stats.currentAmmo;
    }
}

public class PlayerStats : MonoBehaviour
{
    public float health = 100f;
    public int currentAmmo = 30;
}

//TERMINAL INTERACTIVO
using UnityEngine;

public class Terminal : MonoBehaviour, IInteractable
{
    public GameObject doorToUnlock;

    public void Interact()
    {
        doorToUnlock.SetActive(false);
        Debug.Log("Terminal activado: puerta desbloqueada");
    }
}
