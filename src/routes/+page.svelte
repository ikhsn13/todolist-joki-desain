<script>
	import { onMount } from 'svelte'
	import { supabase } from '$lib/supabase'

	let nama = $state('')
	let nomor = $state('')
	let deskripsi = $state('')

	let tasks = $state([])

	let showEdit = $state(false)

	let editId = $state('')
	let editNama = $state('')
	let editNomor = $state('')
	let editDeskripsi = $state('')

	let pending = $derived(tasks.filter((x) => !x.completed))
	let completed = $derived(tasks.filter((x) => x.completed))

	async function loadTasks() {
		const { data, error } = await supabase
			.from('tugas')
			.select('*')
			.order('created_at', { ascending: false })

		if (!error) {
			tasks = data || []
		}
	}

	async function tambah() {
		if (!nama || !nomor) return

		const { error } = await supabase
			.from('tugas')
			.insert([
				{
					nama,
					nomor,
					deskripsi
				}
			])

		if (!error) {
			nama = ''
			nomor = ''
			deskripsi = ''

			await loadTasks()
		}
	}

	function bukaEdit(item) {
		editId = item.id
		editNama = item.nama
		editNomor = item.nomor
		editDeskripsi = item.deskripsi || ''

		showEdit = true
	}

	async function simpanEdit() {
		const { error } = await supabase
			.from('tugas')
			.update({
				nama: editNama,
				nomor: editNomor,
				deskripsi: editDeskripsi
			})
			.eq('id', editId)

		if (!error) {
			showEdit = false
			await loadTasks()
		}
	}

	async function selesai(id) {
		await supabase
			.from('tugas')
			.update({ completed: true })
			.eq('id', id)

		await loadTasks()
	}

	async function hapus(id) {
		await supabase
			.from('tugas')
			.delete()
			.eq('id', id)

		await loadTasks()
	}

	onMount(async () => {
		await loadTasks()
	})
</script>

<div class="container">
	<h1>🎨 ToDoList Joki Desain Grafis</h1>
    <div class="stats">

	<div class="stat-card">
		<div class="stat-number">{tasks.length}</div>
		<div class="stat-label">Total Job</div>
	</div>

	<div class="stat-card">
		<div class="stat-number">{pending.length}</div>
		<div class="stat-label">Aktif</div>
	</div>

	<div class="stat-card">
		<div class="stat-number">{completed.length}</div>
		<div class="stat-label">Selesai</div>
	</div>

</div>

	<div class="grid">

		<!-- CREATE -->
		<div class="card">
			<h2>Create Job</h2>

			<input
				bind:value={nama}
				placeholder="Nama Client"
			/>

			<input
				bind:value={nomor}
				placeholder="Nomor WhatsApp"
			/>

			<textarea
				bind:value={deskripsi}
				placeholder="Deskripsi Pesanan"
			></textarea>

			<button
				class="btn-primary"
				onclick={tambah}
			>
				+ Tambah Job
			</button>
		</div>

		<!-- LIST -->
		<div class="card">
			<h2>List Job ({pending.length})</h2>

			{#if pending.length === 0}
				<p>Belum ada job.</p>
			{/if}

			{#each pending as item}
				<div class="item">

					<b>{item.nama}</b>

					<p>📱 {item.nomor}</p>

					<p>{item.deskripsi}</p>

					<div class="action-group">

						<button
							class="btn-success"
							onclick={() => selesai(item.id)}
						>
							✓ Selesai
						</button>

						<button
							class="btn-primary"
							onclick={() => bukaEdit(item)}
						>
							✏ Edit
						</button>

						<button
							class="btn-danger"
							onclick={() => hapus(item.id)}
						>
							🗑 Hapus
						</button>

					</div>

				</div>
			{/each}
		</div>

		<!-- COMPLETED -->
		<div class="card">
			<h2>Completed ({completed.length})</h2>

			{#if completed.length === 0}
				<p>Belum ada job selesai.</p>
			{/if}

			{#each completed as item}
				<div class="item">

					<b>{item.nama}</b>

					<p>📱 {item.nomor}</p>

					<span class="completed-tag">
						✓ Completed
					</span>

					<div style="margin-top:12px;">
						<button
							class="btn-danger"
							onclick={() => hapus(item.id)}
						>
							🗑 Hapus
						</button>
					</div>

				</div>
			{/each}
		</div>

	</div>
</div>

{#if showEdit}
<div class="modal-overlay">
	<div class="modal-box">

		<h2>Edit Job</h2>

		<input
			bind:value={editNama}
			placeholder="Nama"
		/>

		<input
			bind:value={editNomor}
			placeholder="Nomor"
		/>

		<textarea
			bind:value={editDeskripsi}
			placeholder="Deskripsi"
		></textarea>

		<div class="modal-action">

			<button
				class="btn-primary"
				onclick={simpanEdit}
			>
				Simpan
			</button>

			<button
				class="btn-danger"
				onclick={() => showEdit = false}
			>
				Batal
			</button>

		</div>

	</div>
</div>
{/if}