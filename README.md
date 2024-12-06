<div>
    <x-slot:header>
        <h2 class="font-semibold text-xl text-gray-800 dark:text-gray-200 leading-tight">
            {{ __('User') }}
        </h2>
    </x-slot:header>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white dark:bg-gray-800 overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900 dark:text-gray-100">
                    <livewire:components.table 
                        :model="'App\\Models\\User'"
                        :headers="['name' => 'Name', 'email' => 'Email', 'role' => 'Role']"
                        :fields="[
                            [
                                'type' => 'text',
                                'label' => 'Name',
                                'id' => 'name',
                                'placeholder' => 'Type name',
                                'value' => '',
                                'class' => 'col-span-2',
                                'classNames' => 'bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2.5',
                                'required' => true
                            ],
                            [
                                'type' => 'email',
                                'label' => 'Email',
                                'id' => 'email',
                                'placeholder' => 'Type email',
                                'value' => '',
                                'class' => 'col-span-2',
                                'classNames' => 'bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-primary-600 focus:border-primary-600 block w-full p-2.5',
                                'required' => true
                            ],
                            [
                                'type' => 'select',
                                'label' => 'Role',
                                'id' => 'role',
                                'options' => [
                                    ['value' => '', 'label' => 'Select role'],
                                    ['value' => 'admin', 'label' => 'Admin'],
                                    ['value' => 'user', 'label' => 'User'],
                                ],
                                'value' => '',
                                'class' => 'col-span-2 sm:col-span-1',
                                'classNames' => 'bg-gray-50 border border-gray-300 text-gray-900 text-sm rounded-lg focus:ring-primary-500 focus:border-primary-500 block w-full p-2.5',
                                'required' => true
                            ],
                        ]"
                    />
                </div>
            </div>
        </div>
    </div>
</div>
###
<?php
namespace App\Livewire\Components;

use Livewire\Component;
use Livewire\WithPagination;
use Illuminate\Support\Str;

class Table extends Component
{
    use WithPagination;

    public $model;
    public $searchTerm = '';
    public $perPage = 10;
    public $headers;
    public $fields;
    public $showModal = false;
    public $modalTitle;
    public $modalButtonText;
    public $currentModelId;

    protected $listeners = ['closeModal'];

    public function mount($model, $headers, $fields, $perPage = 10)
    {
        $this->model = $model;
        $this->headers = $headers;
        $this->fields = $fields;
        $this->perPage = $perPage;
    }

    public function updatingSearchTerm()
    {
        $this->resetPage();
    }

    public function openCreateModal()
    {
        $this->modalTitle = 'Create New ' . Str::singular(class_basename($this->model));
        $this->modalButtonText = 'Add new ' . strtolower(Str::singular(class_basename($this->model)));
        $this->currentModelId = null;
        $this->showModal = true;
    }

    public function openEditModal($id)
    {
        $this->currentModelId = $id;
        $modelInstance = $this->model::findOrFail($id);
        $this->modalTitle = 'Edit ' . Str::singular(class_basename($this->model));
        $this->modalButtonText = 'Update ' . strtolower(Str::singular(class_basename($this->model)));
        foreach ($this->fields as &$field) {
            $field['value'] = $modelInstance->{$field['id']} ?? '';
        }
        $this->showModal = true;
    }

    public function submit()
    {
        $data = [];
        foreach ($this->fields as $field) {
            $data[$field['id']] = $this->{$field['id']};
        }

        if ($this->currentModelId) {
            $this->model::find($this->currentModelId)->update($data);
        } else {
            $this->model::create($data);
        }

        $this->closeModal();
    }

    public function delete($id)
    {
        $this->model::findOrFail($id)->delete();
    }

    public function closeModal()
    {
        $this->showModal = false;
    }

    public function render()
    {
        $query = $this->model::query();

        if ($this->searchTerm) {
            foreach ($this->headers as $key => $header) {
                $query->orWhere($key, 'like', '%' . $this->searchTerm . '%');
            }
        }

        $items = $query->paginate($this->perPage);
        return view('livewire.components.table', [
            'items' => $items,
            'total' => $this->model::count(),
        ]);
    }
}
###
<!-- resources/views/livewire/components/table.blade.php -->
<div class="relative overflow-x-auto shadow-md sm:rounded-lg">
    <div class="flex flex-column sm:flex-row flex-wrap space-y-4 sm:space-y-0 items-center justify-between pb-4">
        <button wire:click="openCreateModal"
            class="inline-flex items-center text-gray-500 bg-white border border-gray-300 focus:outline-none hover:bg-gray-100 focus:ring-4 focus:ring-gray-100 font-medium rounded-lg text-sm px-3 py-1.5"
            type="button">
            Create New {{ Str::singular(class_basename($model)) }}
            <svg class="w-2.5 h-2.5 ms-2.5" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none"
                viewBox="0 0 10 6">
                <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                    d="m1 1 4 4 4-4" />
            </svg>
        </button>
        <label for="table-search" class="sr-only">Search</label>
        <div class="relative">
            <div class="absolute inset-y-0 left-0 flex items-center ps-3 pointer-events-none">
                <svg class="w-5 h-5 text-gray-500" aria-hidden="true" fill="currentColor"
                    viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                    <path fill-rule="evenodd"
                        d="M8 4a4 4 0 100 8 4 4 0 000-8zM2 8a6 6 0 1110.89 3.476l4.817 4.817a1 1 0 01-1.414 1.414l-4.816-4.816A6 6 0 012 8z"
                        clip-rule="evenodd"></path>
                </svg>
            </div>
            <input type="text" id="table-search" wire:model="searchTerm"
                class="block p-2 ps-10 text-sm text-gray-900 border border-gray-300 rounded-lg w-80 bg-gray-50 focus:ring-blue-500 focus:border-blue-500"
                placeholder="Search for items">
        </div>
    </div>
    <table class="w-full text-sm text-left text-gray-500">
        <thead class="text-xs text-gray-700 uppercase bg-gray-50">
            <tr>
                <th scope="col" class="p-4">
                    <div class="flex items-center">
                        <input id="checkbox-all-search" type="checkbox"
                            class="w-4 h-4 text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500">
                        <label for="checkbox-all-search" class="sr-only">checkbox</label>
                    </div>
                </th>
                @foreach ($headers as $header)
                    <th scope="col" class="px-6 py-3">{{ $header }}</th>
                @endforeach
                <th scope="col" class="px-6 py-3">Action</th>
            </tr>
        </thead>
        <tbody>
            @foreach ($items as $item)
                <tr class="bg-white border-b dark:bg-gray-800 dark:border-gray-700 hover:bg-gray-50 dark:hover:bg-gray-600">
                    <td class="w-4 p-4">
                        <div class="flex items-center">
                            <input id="{{ $item->id }}" type="checkbox"
                                class="w-4 h-4 text-blue-600 bg-gray-100 border-gray-300 rounded focus:ring-blue-500">
                            <label for="{{ $item->id }}" class="sr-only">checkbox</label>
                        </div>
                    </td>
                    @foreach ($headers as $key => $header)
                        <td class="px-6 py-4">{{ $item->$key }}</td>
                    @endforeach
                    <td class="px-6 py-4 flex gap-x-2">
                        <button wire:click="openEditModal('{{ $item->id }}')" class="font-medium text-yellow-600 hover:underline">Edit</button>
                        <button wire:click="delete('{{ $item->id }}')" class="font-medium text-red-600 hover:underline">Delete</button>
                    </td>
                </tr>
            @endforeach
        </tbody>
    </table>
    {{ $items->links() }}

    <!-- Include FormModal Component -->
    @if ($showModal)
        <livewire:components.form-modal :title="$modalTitle" :fields="$fields" :buttonText="$modalButtonText" :model="$currentModelId" />
    @endif
</div>
<?php

namespace App\Livewire\Components;

use Livewire\Component;

class FormModal extends Component
{
    public $title;
    public $fields;
    public $buttonText;
    public $modelId;
    public $showModal = false;

    protected $listeners = ['openModal', 'closeModal'];

    public function openModal($title, $fields, $buttonText, $modelId = null)
    {
        $this->title = $title;
        $this->fields = $fields;
        $this->buttonText = $buttonText;
        $this->modelId = $modelId;
        $this->showModal = true;
    }

    public function closeModal()
    {
        $this->showModal = false;
    }

    public function submit()
    {
        $data = [];
        foreach ($this->fields as $field) {
            $data[$field['id']] = $this->$field['id'];
        }

        if ($this->modelId) {
            $modelClass = 'App\\Models\\' . $this->title;
            $modelInstance = $modelClass::find($this->modelId);
            $modelInstance->update($data);
        } else {
            $modelClass = 'App\\Models\\' . $this->title;
            $modelClass::create($data);
        }

        $this->closeModal();
        $this->emit('refreshTable');
    }

    public function render()
    {
        return view('livewire.components.form-modal');
    }
}
###
<!-- resources/views/livewire/components/form-modal.blade.php -->
<div>
    @if($showModal)
        <div id="crud-modal" tabindex="-1" aria-hidden="true"
            class="fixed inset-0 z-50 flex items-center justify-center w-full h-full bg-gray-800 bg-opacity-75">
            <div class="relative w-full max-w-md p-4">
                <div class="relative bg-white rounded-lg shadow dark:bg-gray-700">
                    <div class="flex items-center justify-between p-4 md:p-5 border-b rounded-t dark:border-gray-600">
                        <h3 class="text-lg font-semibold text-gray-900 dark:text-white">
                            {{ $title }}
                        </h3>
                        <button type="button" wire:click="closeModal"
                            class="text-gray-400 bg-transparent hover:bg-gray-200 hover:text-gray-900 rounded-lg text-sm w-8 h-8 ms-auto inline-flex justify-center items-center dark:hover:bg-gray-600 dark:hover:text-white">
                            <svg class="w-3 h-3" aria-hidden="true" xmlns="http://www.w3.org/2000/svg" fill="none"
                                viewBox="0 0 14 14">
                                <path stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                                    d="m1 1 6 6m0 0 6 6M7 7l6-6M7 7l-6 6" />
                            </svg>
                            <span class="sr-only">Close modal</span>
                        </button>
                    </div>
                    <form class="p-4 md:p-5" wire:submit.prevent="submit">
                        <div class="grid gap-4 mb-4 grid-cols-2">
                            @foreach ($fields as $field)
                                @if ($field['type'] === 'text' || $field['type'] === 'email' || $field['type'] === 'number' || $field['type'] === 'password')
                                    <livewire:components.form-input :type="$field['type']" :label="$field['label']" :id="$field['id']" :placeholder="$field['placeholder']" :value="$field['value']" :class="$field['class']" :classNames="$field['classNames']" :required="$field['required']" />
                                @elseif ($field['type'] === 'select')
                                    <livewire:components.form-select :label="$field['label']" :id="$field['id']" :options="$field['options']" :value="$field['value']" :class="$field['class']" :classNames="$field['classNames']" :required="$field['required']" />
                                @elseif ($field['type'] === 'textarea')
                                    <livewire:components.form-textarea :label="$field['label']" :id="$field['id']" :placeholder="$field['placeholder']" :value="$field['value']" :class="$field['class']" :classNames="$field['classNames']" />
                                @endif
                            @endforeach
                        </div>
                        <button type="submit"
                            class="text-white inline-flex items-center bg-blue-700 hover:bg-blue-800 focus:ring-4 focus:outline-none focus:ring-blue-300 font-medium rounded-lg text-sm px-5 py-2.5 text-center dark:bg-blue-600 dark:hover:bg-blue-700 dark:focus:ring-blue-800">
                            <svg class="me-1 -ms-1 w-5 h-5" fill="currentColor" viewBox="0 0 20 20" xmlns="http://www.w3.org/2000/svg">
                                <path fill-rule="evenodd" d="M10 5a1 1 0 011 1v3h3a1 1 0 110 2h-3v3a1 1 0 11-2 0v-3H6a1 1 0 110-2h3V6a1 1 0 011-1z" clip-rule="evenodd"></path>
                            </svg>
                            {{ $buttonText }}
                        </button>
                    </form>
                </div>
            </div>
        </div>
    @endif
</div>
